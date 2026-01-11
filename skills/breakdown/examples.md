# Decompose Skill Examples

This file contains complete examples of decompositions across different domains to illustrate the three-layer approach.

---

## Example 1: Technical Architecture - API Rate Limiting

### Layer 1: The Simple Story

Imagine you run a popular restaurant with only one chef. If everyone in town tried to order at once, your chef would be overwhelmed and nothing would get cooked. So you hire a host at the front door whose only job is to say "sorry, we're at capacity right now, please come back in 15 minutes" when too many orders pile up. The host keeps a little notepad tracking how many orders each customer has placed recently, and politely turns away anyone who's been ordering too frequently.

That's exactly what API rate limiting does. Your server is the chef, handling requests (orders). The rate limiter is the host at the door, tracking how many requests each user has made and blocking anyone who's asking for too much too fast. This keeps your server healthy and responsive for everyone, rather than crashing under the weight of one greedy customer or an accidental infinite loop.

```
┌──────────┐
│  Users   │
│ Making   │
│ Requests │
└────┬─────┘
     │
     ▼
┌──────────┐
│   Host   │  ← "Too many orders from you recently!"
│ (Rate    │
│ Limiter) │
└────┬─────┘
     │
     ▼
┌──────────┐
│   Chef   │
│ (Server) │
└──────────┘
```

---

### Layer 2: The Architecture

**Big Picture**: Just like our restaurant host uses a notepad to track recent orders, the rate limiter uses a data structure (called a sliding window counter or token bucket) to track request counts per user. When a request arrives, the limiter checks this counter before deciding whether to forward it to your application server or reject it with a "429 Too Many Requests" response.

**Request Flow**:
- Incoming request hits the rate limiter first (before reaching your app)
- Limiter extracts user identifier (API key, IP address, or user ID from auth token)
- Checks counter: "Has this user exceeded their quota in the current time window?"
- **Accept path**: Counter is OK → increment count → forward request → return response
- **Reject path**: Quota exceeded → return 429 status → include "Retry-After" header

**Storage & State**:
- Use fast in-memory store like Redis (disk would be too slow for every request)
- Key format: `ratelimit:{user_id}:{time_bucket}`
- Store request count and window start time
- Set TTL (time-to-live) so old data expires automatically

**Configuration**:
- Rate limit rules per endpoint or per user tier (free users: 100/hour, paid: 10,000/hour)
- Time window choices: fixed window (simple but has edge case bursts), sliding window (smoother), token bucket (allows bursts up to a limit)
- Enforcement location: API gateway (protects everything), application middleware (more flexible per-route limits)

```
┌────────────────────────────────────────┐
│         Incoming Request               │
└───────────────┬────────────────────────┘
                │
                ▼
       ┌────────────────┐
       │  Extract User  │
       │   Identifier   │
       └────────┬───────┘
                │
                ▼
       ┌────────────────┐
       │  Check Redis   │──→ Key: ratelimit:user123:1640000000
       │    Counter     │     Value: {count: 47, limit: 100}
       └────────┬───────┘
                │
          ┌─────┴─────┐
          ▼           ▼
    ┌─────────┐  ┌─────────┐
    │ Allowed │  │ Blocked │
    │ (< 100) │  │ (≥ 100) │
    └────┬────┘  └────┬────┘
         │            │
         │            └──→ Return 429 + Retry-After: 3600
         │
         ▼
    ┌─────────┐
    │Increment│
    │ Counter │
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │ Forward │
    │   to    │
    │  Server │
    └─────────┘
```

---

### Layer 3: The Complete Blueprint

### Request Processing Pipeline

**Purpose**: Handle each incoming request through the rate limiting check before application logic runs.

- **Entry point**: Middleware or reverse proxy (Nginx, API Gateway, application middleware)
- **Identifier extraction**:
  - Priority order: API key (from `Authorization` header) → User ID (from JWT claims) → IP address (fallback)
  - Hash identifier to fixed length for consistent Redis key size
  - Handle edge cases: missing identifier defaults to IP, localhost gets special high limit
- **Redis interaction**:
  - Command: `INCR ratelimit:{hashed_id}:{time_bucket}` (atomic increment)
  - If key doesn't exist: Redis creates it with value 1, set `EXPIRE {ttl_seconds}`
  - Read response: counter value after increment
- **Decision logic**:
  ```python
  if counter <= limit:
      return ALLOW, remaining=(limit - counter)
  else:
      return DENY, retry_after=(time_until_next_window)
  ```
- **Response enrichment**:
  - Add headers to all responses (success or blocked):
    - `X-RateLimit-Limit: 100`
    - `X-RateLimit-Remaining: 53`
    - `X-RateLimit-Reset: 1640003600` (Unix timestamp)
  - 429 response body: `{"error": "rate_limit_exceeded", "retry_after_seconds": 3600}`

### Time Window Algorithms

**Fixed Window**:
- Bucket: `floor(current_time / window_size)`
- Example: 1:00-1:59pm is bucket `1640000000`
- Pros: Simple, low memory (one counter per window)
- Cons: Edge case burst (user makes 100 requests at 1:59pm, then 100 more at 2:00pm = 200 in 1 minute)

**Sliding Window**:
- Weighted count from two buckets: `(prev_count * (1 - time_in_current)) + current_count`
- Example at 1:30pm: `(59_requests * 0.5) + 30_requests = 59.5` (round up to 60)
- Pros: Smoother rate limiting, no edge case bursts
- Cons: Slightly more complex calculation

**Token Bucket** (recommended):
- Tokens regenerate at fixed rate (e.g., 100 tokens/hour = 1.67 tokens/minute)
- Each request consumes 1 token
- Allow burst up to bucket capacity (e.g., capacity=120 allows 20-request burst)
- Redis data: `{tokens: 87.3, last_refill: 1640001234}`
- Refill logic on each request:
  ```python
  elapsed = now - last_refill
  new_tokens = min(capacity, current_tokens + (rate * elapsed))
  ```
- Pros: Allows natural bursts, intuitive, smooth over time
- Cons: Requires float arithmetic (slightly slower)

### Storage Architecture

**Redis Setup**:
- **Deployment**: Redis Cluster (3 master + 3 replica) for high availability
- **Key namespace**: `ratelimit:v2:{user_id}:{endpoint}:{bucket}`
  - Version in key allows migration to new algorithm
  - Endpoint granularity: `/api/upload` has stricter limit than `/api/read`
- **Memory optimization**:
  - Use Redis hashes for multiple counters per user: `HINCRBY ratelimit:user123 /api/upload 1`
  - Set TTL = 2 × window_size (keeps previous window for sliding calculation)
  - Estimated memory: 50 bytes per active user per endpoint = 5MB for 100K users
- **Failover behavior**:
  - If Redis unavailable: **fail open** (allow requests) or **fail closed** (deny all)?
  - Recommendation: Fail open for grace, use circuit breaker to prevent Redis stampede

### Configuration Schema

```yaml
rate_limits:
  - name: free_tier
    identifiers: [api_key]
    rules:
      - endpoint: /api/*
        limit: 100
        window: 3600  # 1 hour
        algorithm: token_bucket
      - endpoint: /api/upload
        limit: 10
        window: 3600  # stricter for expensive ops

  - name: paid_tier
    identifiers: [api_key]
    rules:
      - endpoint: /api/*
        limit: 10000
        window: 3600
        burst_capacity: 12000  # allow 20% burst

  - name: ip_fallback
    identifiers: [ip_address]
    rules:
      - endpoint: /api/*
        limit: 50
        window: 3600
```

### Error Handling & Edge Cases

- **Clock skew**: Use Redis server time (`TIME` command) not application server time
- **Distributed deployment**: All app servers must share same Redis cluster (no local caching of counts)
- **Key expiration race**: User at exactly 99 requests, key expires, next request sees count=1 instead of 100
  - Solution: Check key TTL, if < 50% of window remaining, don't reset count
- **Negative remaining**: If limit changed from 100→50 mid-window, remaining could show negative
  - Display as 0, don't reject until next window
- **Thundering herd**: Many users hitting limit at window boundary
  - Add jitter to retry-after: `retry_after = base_time + random(0, 60)`

### Monitoring & Observability

- **Metrics to track**:
  - `rate_limit_blocks_total` (counter, labels: endpoint, tier)
  - `rate_limit_remaining_avg` (gauge, histogram)
  - `redis_latency_ms` (histogram, p50/p95/p99)
- **Alerts**:
  - `rate_limit_blocks > 10% of requests` for 5 min → Limits too strict?
  - Redis latency p99 > 10ms → Performance degradation
  - Redis connection errors → Failover issue
- **Logging**: Log 429 responses with user_id, endpoint, count/limit for analysis

### Testing Strategy

- **Unit tests**: Each algorithm (fixed, sliding, token bucket) with mocked Redis
- **Integration tests**: Real Redis instance, test window boundaries, expiration, concurrent requests
- **Load tests**: 10K req/sec with 1000 unique users, verify no count leakage between users
- **Chaos tests**: Kill Redis mid-request, verify graceful degradation

```
┌──────────────────────────────────────────────────────────────┐
│                    Incoming Request                          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │  API Gateway /       │
              │  Reverse Proxy       │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ Extract Identifier   │
              │ • API Key (header)   │
              │ • User ID (JWT)      │
              │ • IP Address (conn)  │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │  Hash ID + Build Key │
              │  ratelimit:v2:{id}   │
              └──────────┬───────────┘
                         │
         ┌───────────────┴──────────────┐
         │                              │
         ▼                              ▼
┌─────────────────┐          ┌─────────────────┐
│  Redis Cluster  │          │  Config Store   │
│  (Token Bucket) │          │  (Limits/Rules) │
│                 │          │                 │
│ Master 1 ┬──────┼─────────→│ - Tiers        │
│ Master 2 │      │          │ - Endpoints     │
│ Master 3 │      │          │ - Algorithms    │
└──────────┼──────┘          └─────────────────┘
           │
  ┌────────┴─────────┐
  │ INCR or HINCRBY  │
  │ Get: tokens, TTL │
  └────────┬─────────┘
           │
       ┌───┴─────┐
       │Calculate│
       │ tokens  │
       │remaining│
       └───┬─────┘
           │
      ┌────┴─────┐
      ▼          ▼
 ┌────────┐  ┌────────┐
 │ ALLOW  │  │  DENY  │
 │tokens≥1│  │tokens<1│
 └───┬────┘  └────┬───┘
     │            │
     │            └──→ ┌─────────────────────┐
     │                 │ Return 429          │
     │                 │ X-RateLimit-*       │
     │                 │ Retry-After: {sec}  │
     │                 └─────────────────────┘
     │
     ▼
┌─────────────────┐
│ Decrement Token │
│ Update last_req │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Forward to App  │
│ + Add Headers:  │
│ X-RateLimit-*   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐       ┌──────────────────┐
│ Application     │       │  Observability   │
│ Server Logic    │──────→│  • Metrics       │
└────────┬────────┘       │  • Logs          │
         │                │  • Traces        │
         ▼                └──────────────────┘
┌─────────────────┐
│ Return Response │
│ (with headers)  │
└─────────────────┘

Legend:
─────→  Data flow
──────  Grouping/Container
```

---

## Example 2: Business Plan - Launching a SaaS Product

### Layer 1: The Simple Story

Think of launching a SaaS product like planning a road trip. First, you need to know where you're going (who's your customer, what problem are you solving?). Then you need to figure out your route - you can't drive straight from New York to LA through mountains and rivers, you need roads that actually exist (your go-to-market strategy). You'll need gas money for the trip (capital for runway), and you need to know where the rest stops are (milestones that tell you you're on track).

Most importantly, you need passengers who want to go where you're going (early adopters who have the problem you're solving). If you just build a beautiful bus but park it in your garage, you'll run out of gas money before anyone gets on board. The trick is to start with a small, reliable shuttle service for people desperately trying to get somewhere (solve one painful problem really well), then expand your routes as you earn money from those first passengers.

```
┌──────────┐
│  Problem │
│ Discovery│
└─────┬────┘
      │
      ▼
┌──────────┐     ┌──────────┐
│  Build   │────→│ Get Early│
│ Shuttle  │     │ Riders   │
└──────────┘     └─────┬────┘
                       │
                       ▼
                 ┌──────────┐
                 │ Expand   │
                 │ Routes   │
                 └──────────┘
```

---

### Layer 2: The Architecture

**Big Picture**: Like the road trip, a SaaS launch follows a sequence of connected phases. You start with customer discovery (finding people with a painful problem worth paying to solve), then build your MVP (the "shuttle service"), launch to a small group of early adopters, validate that they see value (product-market fit), then scale up marketing and sales. Each phase has different goals and metrics.

**Customer Discovery & Validation**:
- Identify target segment: specific job title, company size, industry (e.g., "HR managers at 50-200 person tech companies")
- Conduct 20-30 problem interviews before building anything
- Look for "hair on fire" moments - problems people are already trying to solve with painful workarounds
- Validate willingness to pay: "If this solved your problem, what would it be worth to you?"

**MVP Development**:
- Core feature set: solve ONE problem extremely well (not ten problems poorly)
- Technical scope: focus on functionality over polish - early adopters forgive ugly UX if it solves their pain
- Timeline: 6-8 weeks from start to first paying customer (not "6 months to perfect product")
- Pricing strategy: charge from day one (even if it's $50/month) to validate willingness to pay

**Go-to-Market Strategy**:
- Early adopter channels: where do your target customers already congregate? (Slack communities, LinkedIn groups, subreddits)
- Founder-led sales initially (you need to hear objections firsthand, not through a sales rep)
- Content marketing: share learnings, build authority, attract inbound leads
- Referral mechanics: make it easy for happy customers to recommend you

**Growth & Scaling**:
- Product-market fit signal: organic word-of-mouth, low churn (<5%/month), customers pulling you toward features
- Hire order: customer success first (retain customers), sales second (new customers), marketing third (scale acquisition)
- Capital efficiency: can you reach $10K MRR on <$50K spent? Proves unit economics before raising money

```
┌─────────────────────────────────────────┐
│       PHASE 1: Discovery (Weeks 1-4)    │
│  ┌──────────┐      ┌──────────┐        │
│  │ 30 inter-│─────→│ Problem  │        │
│  │  views   │      │ validated│        │
│  └──────────┘      └──────────┘        │
└──────────────────────┬──────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────┐
│     PHASE 2: Build MVP (Weeks 5-10)     │
│  ┌──────────┐      ┌──────────┐        │
│  │ Core     │─────→│ Beta w/  │        │
│  │ features │      │ 5 users  │        │
│  └──────────┘      └──────────┘        │
└──────────────────────┬──────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────┐
│    PHASE 3: Launch (Weeks 11-16)        │
│  ┌──────────┐      ┌──────────┐        │
│  │ Founder- │─────→│ 10 paying│        │
│  │ led sales│      │ customers│        │
│  └──────────┘      └──────────┘        │
└──────────────────────┬──────────────────┘
                       │
                  ┌────┴────┐
                  │ PMF ?   │
                  └────┬────┘
                       │
                 ┌─────┴─────┐
                 ▼           ▼
          ┌──────────┐  ┌──────────┐
          │ Scale    │  │ Pivot/   │
          │ (hire,   │  │ Iterate  │
          │ market)  │  │          │
          └──────────┘  └──────────┘
```

---

### Layer 3: The Complete Blueprint

### Phase 1: Customer Discovery & Validation (Weeks 1-4)

**Purpose**: Validate that a specific segment has a painful problem worth building a solution for, before writing any code.

**Target Segment Definition**:
- **Demographics**: Job title (HR Manager, VP of Sales), company size (50-200 employees), industry (B2B SaaS), geography (US-based initially)
- **Psychographics**: Uses which tools already? (Greenhouse, Lever for recruiting), Values what? (efficiency, compliance)
- **Sample size**: Need 30 interviews to see patterns, 20+ saying "I would pay for this" to validate
- **Where to find them**: LinkedIn outreach, industry Slack communities, warm intros from your network

**Interview Script & Process**:
- **Don't pitch, just listen**: "Tell me about the last time you [struggled with X problem]"
- **Understand current behavior**: "What do you do today to solve this?" (reveals pain level - using 3 tools + spreadsheet = high pain)
- **Identify willingness to pay**: "If a tool could [do Y], what would be a no-brainer price?" then "What would be expensive but you'd still consider?"
- **Red flags**: They say "that would be nice to have" (not painful enough), or "my assistant handles that" (not the buyer)
- **Green flags**: They lean forward, ask "when can I get this?", describe workarounds in detail (high pain)
- **Capture quotes**: Record (with permission) specific phrases for landing page copy later

**Validation Criteria**:
- 20+ interviews where prospect says "I would pay for this"
- Consistent problem framing (if 30 people describe 30 different problems, segment too broad)
- Willingness to pay ≥ $50/month/user (validates B2B SaaS business model)
- At least 5 people willing to be design partners (beta test for free, give feedback)

**Deliverable**: One-page problem/solution brief with quotes, target segment definition, pricing hypothesis

---

### Phase 2: MVP Development (Weeks 5-10)

**Purpose**: Build the minimum feature set that solves the core problem for early adopters, optimizing for speed and validation (not perfection).

**Core Feature Scoping**:
- Start with job-to-be-done: "When [situation], I want to [motivation], so I can [outcome]"
- Example: "When hiring for a new role, I want to quickly see if a candidate has the required skills, so I can focus interview time on cultural fit"
- **In scope**: Features directly solving that one job (skill parsing from resume, yes/no match against job req)
- **Out of scope**: Nice-to-haves (interview scheduling, offer letter templates, reporting dashboards)
- **Tech stack choices**: Use what you know (don't learn a new framework for MVP), managed services over self-hosted (Firebase > self-hosted DB)

**Development Sprint Structure**:
- Week 5-6: Core data model + API routes (get one workflow end-to-end working)
- Week 7-8: Basic UI (functional, not beautiful - use component library like Tailwind)
- Week 9: Beta test with 5 design partners, fix critical bugs only
- Week 10: Onboarding flow + payment integration (Stripe Checkout)

**Design Partner Program**:
- Recruit 5 people from discovery interviews who said "I want early access"
- Free access in exchange for 30-min feedback session every week
- Give them tasks: "Upload 3 resumes and tell me what's confusing"
- Prioritize bugs: P0 = blocks core workflow, P1 = annoying but workaround exists, P2 = polish
- Ship P0 fixes within 24 hours, batch P1/P2 for weekly release

**Launch Readiness Checklist**:
- [ ] One workflow works end-to-end (resume upload → skill match → decision)
- [ ] 3/5 design partners say "I would pay for this" after using it
- [ ] Payment integration live (Stripe Checkout or Paddle)
- [ ] Basic landing page with value prop + pricing
- [ ] Support email set up (even if just Inbox)

---

### Phase 3: Go-to-Market & Early Customer Acquisition (Weeks 11-20)

**Purpose**: Acquire first 10-20 paying customers through founder-led sales, validate pricing, iterate based on feedback.

**Founder-Led Sales Process**:
- **Lead sources**: (1) Design partners converting to paid, (2) Re-engage discovery interviewees, (3) Warm intros from advisors/investors, (4) Cold LinkedIn outreach (10-20/day)
- **Outreach template**: "Hi [Name], I interviewed you 3 months ago about [problem]. We've built an MVP and have 5 companies using it. Would love 15 min to show you how [Company X] is using it."
- **Demo format**: 15-min Zoom, 5 min context ("what's your process today?"), 7 min demo (show them THEIR use case, not generic), 3 min Q&A
- **Closing**: "Does this solve your problem? Our pricing is $X/month. Want to start a trial?" (always ask for the sale, even if it feels pushy - you need feedback)
- **Objection handling**:
  - "Too expensive" → "What's a no-brainer price?" (find price sensitivity)
  - "Need to think about it" → "What concerns do you have?" (uncover real objection)
  - "Need more features" → "Which feature would make this a must-have?" (prioritization signal)

**Pricing Strategy**:
- **Starting hypothesis**: $99/month for 5 users (from discovery interviews)
- **Test willingness to pay**: If 8/10 say yes immediately, price too low; if 2/10 say yes, too high; 5/10 = sweet spot
- **Annual vs monthly**: Offer annual discount (2 months free) to improve cash flow
- **Add-ons**: Keep simple - one core plan, maybe "Enterprise" tier for custom features

**Content Marketing Engine**:
- **Goal**: Attract inbound leads via search + social
- **Topics**: Share learnings from customer interviews, how customers use the product, industry insights
- **Channels**: LinkedIn posts (3x/week), blog on your site (1x/week), industry newsletter sponsorships
- **Metrics**: Track traffic → email signups → trial starts → paid conversions

**Metrics Dashboard** (review weekly):
- **Acquisition**: Demos booked, trial starts, paid conversions (goal: 30% demo → paid)
- **Activation**: % of trials that complete core workflow in first week (goal: >70%)
- **Revenue**: MRR (monthly recurring revenue), growth rate (goal: 20% MoM)
- **Retention**: Churn rate (goal: <5%/month), NPS (goal: >40)

---

### Phase 4: Product-Market Fit Validation & Scaling (Weeks 21-40)

**Purpose**: Confirm you have PMF, then scale acquisition and build the team.

**PMF Signals** (need 3+ to be confident):
1. **Organic growth**: Customers refer others without prompting (viral coefficient >0.2)
2. **Low churn**: <5% monthly churn (customers stick around)
3. **Usage frequency**: Core workflow used 3+ times/week (high engagement)
4. **"Very disappointed" score**: >40% say they'd be "very disappointed" if product went away (Sean Ellis test)
5. **Inbound demand**: You're turning down leads because too busy

**Scaling Acquisition**:
- **Hire first salesperson** when you have 10+ paying customers and repeatable demo-to-close process
- **Paid ads** (Google, LinkedIn) when CAC (customer acquisition cost) < 3 month LTV (lifetime value)
- **Partnerships**: Integrate with tools your customers already use (e.g., Greenhouse API if building recruiting tool)

**Team Hiring Priority**:
1. **Customer Success** (first hire): Onboarding, support, retention (frees founder to sell)
2. **Sales** (second hire): Close inbound leads, cold outreach (scales acquisition)
3. **Engineer** (third hire): Ship features faster, reduce founder dev workload
4. **Marketing** (fourth hire): Content, SEO, paid ads (scales inbound)

**Capital Requirements & Runway**:
- **Bootstrap scenario**: Reach $10K MRR on <$50K spent (founder salary + SaaS tools + ads), then default alive (revenue > expenses)
- **Seed scenario**: Raise $500K-$1M at $10K MRR to accelerate hiring, target $100K MRR in 12 months
- **Burn rate**: Aim for 12-18 month runway (e.g., $500K raise = max $35K/month burn)

---

### Risk Mitigation & Contingencies

**Risk: No one wants to pay**
- **Early signal**: <20% of demos convert to paid trials
- **Mitigation**: Re-examine pricing (too high?), validate problem is painful enough (maybe "nice to have"), improve demo (show clear ROI)
- **Pivot decision**: If <5 paying customers after 20 weeks, re-do customer discovery or pivot to different segment

**Risk: High churn (customers cancel quickly)**
- **Early signal**: >10% monthly churn
- **Mitigation**: Conduct exit interviews ("why did you cancel?"), improve onboarding (do they complete core workflow?), add missing critical features
- **Threshold**: If churn doesn't drop below 7% after 3 months of iteration, product may not be solving problem well enough

**Risk: Can't acquire customers cheaply enough**
- **Early signal**: CAC (cost to acquire) > 12 month LTV
- **Mitigation**: Focus on organic channels (content, referrals), improve sales conversion rate, raise prices to increase LTV
- **Threshold**: If CAC:LTV ratio doesn't improve to 1:3 within 6 months, business model may not be viable

**Risk: Competitor launches similar product**
- **Mitigation**: Speed to market (why 6-week MVP critical), build relationships with early customers (switching cost), focus on specific niche they ignore
- **Competitive moat**: Network effects (if possible), deep integrations, or specialized domain expertise

---

### Timeline & Milestones Gantt Chart

```
Week  Phase            Milestones                     Team      Spend
────────────────────────────────────────────────────────────────────
1-4   Discovery        30 interviews                  Founder   $2K
                       20 validated                            (tools)
                       5 design partners

5-10  MVP Build        Core feature done              Founder   $5K
                       5 beta users testing                    (dev tools,
                       Payment integration                     hosting)

11-16 Launch           10 paying customers            Founder   $8K
                       $1K MRR                                 (ads, tools)

17-20 Iterate          15 paying customers            Founder   $8K
                       $2K MRR                                 (ads)
                       <8% churn

21-24 PMF Validation   25 customers                   +CS hire  $15K
                       $4K MRR                        (2 ppl)   (salary,
                       3+ PMF signals                          ads)

25-32 Scale            50 customers                   +Sales    $30K/mo
                       $10K MRR                       (3 ppl)
                       Seed raise ($500K)

33-40 Growth           100 customers                  +Eng      $40K/mo
                       $20K MRR                       +Mktg
                       20% MoM growth                 (5 ppl)
────────────────────────────────────────────────────────────────────
Total runway: 40 weeks | Total spend: ~$250K to reach $20K MRR
```

---

### Detailed System Diagram

```
┌───────────────────────────────────────────────────────────────────┐
│                         MARKET & CUSTOMERS                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Target       │  │ Early        │  │ Scaling      │          │
│  │ Segment      │─→│ Adopters     │─→│ Mainstream   │          │
│  │ (interviews) │  │ (first 20)   │  │ (100+)       │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└───────────────────────────┬───────────────────────────────────────┘
                            │
                    ┌───────┴────────┐
                    │  Acquisition   │
                    │   Channels     │
                    └───────┬────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐  ┌────────────────┐  ┌───────────────┐
│ Founder-Led   │  │ Content        │  │ Paid Ads      │
│ Outreach      │  │ Marketing      │  │ (later stage) │
│ • LinkedIn    │  │ • Blog         │  │ • Google      │
│ • Warm intros │  │ • LinkedIn     │  │ • LinkedIn    │
│ • Re-engage   │  │ • SEO          │  │ • CAC < 3mo   │
│   interviews  │  │ • Newsletter   │  │   LTV         │
└───────┬───────┘  └────────┬───────┘  └───────┬───────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   Landing Page +      │
                │   Trial Signup        │
                └───────────┬───────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   Product (MVP)       │
                │ ┌─────────────────┐   │
                │ │ Onboarding Flow │   │
                │ │ • First 5 min   │   │
                │ │ • Aha moment    │   │
                │ └────────┬────────┘   │
                │          │            │
                │          ▼            │
                │ ┌─────────────────┐   │
                │ │  Core Workflow  │   │
                │ │ • Job-to-be-done│   │
                │ │ • 1 problem     │   │
                │ └────────┬────────┘   │
                │          │            │
                │          ▼            │
                │ ┌─────────────────┐   │
                │ │  Payment Gate   │   │
                │ │ • Stripe        │   │
                │ │ • $99/mo        │   │
                │ └─────────────────┘   │
                └───────────┬───────────┘
                            │
                    ┌───────┴────────┐
                    │  Paying        │
                    │  Customer      │
                    └───────┬────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐  ┌────────────────┐  ┌───────────────┐
│ Success &     │  │ Usage &        │  │ Expansion     │
│ Support       │  │ Engagement     │  │ Revenue       │
│ • Onboarding  │  │ • Track key    │  │ • Upsell      │
│ • Help desk   │  │   metrics      │  │ • Referrals   │
│ • Health score│  │ • 3x/week use? │  │ • Annual plan │
└───────┬───────┘  └────────┬───────┘  └───────┬───────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  Retention    │
                    │  <5% churn    │
                    └───────┬───────┘
                            │
                ┌───────────┴──────────┐
                ▼                      ▼
        ┌───────────────┐      ┌──────────────┐
        │ PMF Signals   │      │ Churn        │
        │ • Organic ref │      │ • Exit inter │
        │ • NPS >40     │      │ • Win-back   │
        │ • Usage high  │      │              │
        └───────┬───────┘      └──────────────┘
                │
                ▼
        ┌───────────────┐
        │ Scale Phase   │
        │ • Hire team   │
        │ • Raise $     │
        │ • Paid acq    │
        └───────────────┘


┌─────────────────────────────────────────────────────────────┐
│                      KEY METRICS DASHBOARD                  │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│ │ MRR          │ │ Churn Rate   │ │ CAC:LTV      │        │
│ │ (goal: 20%   │ │ (goal: <5%)  │ │ (goal: 1:3)  │        │
│ │  MoM growth) │ │              │ │              │        │
│ └──────────────┘ └──────────────┘ └──────────────┘        │
│                                                             │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│ │ Trial→Paid   │ │ NPS          │ │ Runway       │        │
│ │ (goal: 30%)  │ │ (goal: >40)  │ │ (months left)│        │
│ └──────────────┘ └──────────────┘ └──────────────┘        │
└─────────────────────────────────────────────────────────────┘

Legend:
─────→  Process flow
──────  Grouping/System
```

