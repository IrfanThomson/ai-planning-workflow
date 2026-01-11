# Brainstorm Skill Examples

This file contains complete examples of brainstorming sessions across different domains to illustrate the five-phase process.

---

## Example 1: Technical Problem - Reducing API Response Time

**User Request**: "Our `/api/dashboard` endpoint is taking 3-5 seconds to load. Brainstorm ways to make it faster."

---

### Phase 1: Problem Framing

**Problem Restatements**:
- **How Might We**: How might we deliver dashboard data to users in under 500ms?
- **Pain to Eliminate**: Users are frustrated waiting 3-5 seconds for the dashboard to load, causing them to refresh repeatedly
- **Outcome to Achieve**: Dashboard loads feel instant (< 500ms perceived load time)
- **Constraint to Remove**: The constraint is that we're computing everything on-demand with no caching

**Success Criteria**:
- **Must-haves**:
  - Response time < 1 second (p95)
  - Data must be fresh (no stale data > 5 minutes old)
  - No breaking changes to API contract
- **Nice-to-haves**:
  - Real-time updates (no refresh needed)
  - Costs don't increase >20%
  - Works for all users (not just premium tier)
- **Evaluation dimensions**: Impact (user experience improvement), Feasibility (engineering effort), Speed (time to ship), Cost (infrastructure spend)

**Assumptions to Challenge**:
- We assume users need all data at once
- We assume we can't cache because data changes frequently
- We assume we need to query 8 different tables
- We assume users care about seeing data from the last 5 minutes
- We assume the frontend needs raw data (not pre-aggregated)

---

### Phase 2: Divergent Exploration

#### SCAMPER Ideas (7 ideas)

1. **Substitute**: What if we replaced synchronous DB queries with a pre-aggregated read replica updated every minute?
2. **Combine**: What if we combined multiple small queries into one giant SQL JOIN with proper indexes?
3. **Adapt**: How do analytics dashboards handle this? They use data warehouses with materialized views - could we do that?
4. **Modify**: What if we made the queries asynchronous and showed a loading skeleton, populating sections as data arrives?
5. **Put to other use**: What if we repurposed our background job queue to pre-compute dashboard data on a schedule?
6. **Eliminate**: What if we removed 50% of the dashboard widgets and only showed what users actually look at (based on analytics)?
7. **Reverse**: What if instead of the server computing on request, the frontend computed from raw events streamed in real-time?

#### First Principles (5 ideas)

8. Strip to core: The fundamental problem is "fetch + aggregate + serialize 100K rows every request." What if we only fetch the delta since last request?
9. Challenge assumption: Do users really need 5-minute freshness? Survey shows most check dashboard once/hour. Cache for 15 minutes.
10. Build from scratch: If we had no legacy DB schema, we'd store dashboard data denormalized in Redis with TTL. Can we add Redis alongside Postgres?
11. What if we could: What if we could know which users will request the dashboard next? Pre-warm cache for active users based on session activity.
12. Rethink the metric: "Response time" is wrong metric. Users care about "time to useful data." Show critical widgets first (above fold), lazy-load rest.

#### Analogical Thinking (6 ideas)

13. **From video streaming**: Netflix doesn't load the whole movie upfront - they stream chunks. Stream dashboard data in chunks (critical data first, then enrich).
14. **From video games**: Games pre-load assets based on player location. Pre-compute dashboard data for users likely to visit (active sessions, time-of-day patterns).
15. **From restaurants**: Restaurants prep ingredients before dinner rush (mise en place). Pre-compute aggregates during low-traffic hours (2-5am).
16. **From newspapers**: Print newspapers are stale but fast. Digital is fresh but slow. Hybrid: cached snapshot + real-time diff overlay.
17. **From CDNs**: CDNs cache at edge nodes near users. What if we cached dashboard data regionally in Redis clusters close to user geo-locations?
18. **From compilers**: Compilers cache intermediate build artifacts. Cache intermediate query results (user list, project stats) and combine them.

#### Constraint Removal (6 ideas)

19. **Unlimited budget**: Spin up dedicated read replicas per region + Redis cluster + elasticsearch for analytics. Elasticsearch aggregations are blazing fast.
20. **Unlimited time**: Rewrite entire DB schema to event-sourced architecture. Every change is an event. Dashboard reads from materialized view updated by event stream.
21. **Perfect data**: If we had perfect prediction of which users visit when, we'd pre-compute and warm cache 30 seconds before they click. Use ML model to predict.
22. **No regulations**: If we could store user data anywhere, we'd replicate to edge Cloudflare Workers KV and serve from there (sub-100ms globally).
23. **10% version of #20**: Don't rewrite everything, but introduce an event log for the 3 highest-traffic tables. Materialize dashboard view from events.
24. **Adjacent win from #19**: We can't afford Elasticsearch, but we can add a single Redis instance. Cache the top 3 slowest queries (identified by profiling).

#### Inversion & Provocation (6 ideas)

25. **Make it worse, then invert**: Make it slower by adding more queries → Invert: Remove queries. Turns out 5 of 8 queries are for widgets 90% of users never open. Remove them.
26. **Stupidest idea**: Show completely fake data instantly, then replace with real data. Insight: Users don't verify every number - can we show stale data with "Updated 2 min ago" label?
27. **What competitors won't do**: Competitors optimize for freshness. We optimize for speed and show "Last updated: X min ago" timestamp. Users can manually refresh if they need fresh data.
28. **Opposite of obvious**: Instead of making server faster, make client slower (add 200ms delay) so user perceives 1s as fast by comparison. (Jk, but... anchor expectations with progress bar?)
29. **Provocative reframe**: The problem isn't slow API. It's that users refresh too often because the UI doesn't tell them when data is fresh. Add websocket that pushes "new data available" notification.
30. **Wildcard**: What if dashboard was a static site generated every 5 minutes and served from S3/CDN? Works for 80% of users who don't need real-time. Power users get real-time version.

**Total**: 30 raw ideas

---

### Phase 3: Clustering & Combination

#### Cluster 1: Caching Strategies
- **Ideas**: #1 (read replica), #9 (15-min cache), #10 (Redis), #24 (cache top 3 queries)
- **Core approach**: Reduce DB load by caching computed results with acceptable staleness

#### Cluster 2: Pre-computation
- **Ideas**: #5 (background job), #14 (pre-load for likely users), #15 (prep during off-hours), #23 (event log + materialized view)
- **Core approach**: Compute dashboard data before users request it (push model instead of pull)

#### Cluster 3: Progressive Loading
- **Ideas**: #4 (async queries + skeleton), #12 (critical data first), #13 (stream in chunks), #16 (snapshot + real-time diff)
- **Core approach**: Show something useful immediately, then enhance with more data

#### Cluster 4: Data Reduction
- **Ideas**: #6 (remove unused widgets), #8 (fetch delta only), #25 (remove unnecessary queries)
- **Core approach**: Send less data by eliminating what users don't need

#### Cluster 5: Edge Distribution
- **Ideas**: #17 (regional Redis), #19 (Elasticsearch), #22 (edge workers), #30 (static site from CDN)
- **Core approach**: Move data closer to users geographically

#### Cluster 6: Client-Side Approaches
- **Ideas**: #7 (frontend computes from events), #26 (stale data + label), #27 (manual refresh), #29 (websocket notifications)
- **Core approach**: Shift some work to the client or set user expectations differently

#### Hybrid Combinations

- **Hybrid A: Smart Cache + Progressive Load**: Combine #10 (Redis cache) + #12 (critical data first). Cache full dashboard in Redis (15-min TTL), but API returns critical widgets immediately from cache, then streams remaining widgets asynchronously.

- **Hybrid B: Pre-compute + Edge Cache**: Combine #5 (background job) + #17 (regional Redis). Background job pre-computes dashboard every 5 minutes, writes to Redis cluster. API reads from regional Redis (sub-50ms latency).

- **Hybrid C: Reduce + Cache**: Combine #6 (remove unused widgets) + #24 (cache top 3). First, analyze which widgets users actually click (remove the rest). Then cache the remaining 3-5 expensive queries in Redis.

#### Standout Outliers

- **Outlier #21 (ML prediction)**: Using ML to predict which users will request dashboard next is over-engineered for this problem, but interesting for future exploration if we build a general-purpose pre-computation system.

- **Outlier #30 (static site)**: Radical idea - dashboard as static site regenerated every 5 min. Doesn't fit any cluster but could be perfect for certain use cases (public dashboards, internal metrics).

---

### Phase 4: Convergent Evaluation

| Concept | Impact | Feasibility | Speed | Cost | Total | Notes |
|---------|--------|-------------|-------|------|-------|-------|
| Cluster 1: Caching | 4 | 5 | 5 | 4 | 18 | High feasibility, quick win, proven approach |
| Cluster 2: Pre-computation | 5 | 3 | 3 | 3 | 14 | High impact but requires background job infrastructure |
| Cluster 3: Progressive Load | 3 | 4 | 4 | 5 | 16 | Good UX improvement, doesn't solve root problem |
| Cluster 4: Data Reduction | 4 | 5 | 5 | 5 | 19 | Easiest win, no infra cost, requires product decision |
| Cluster 5: Edge Distribution | 5 | 2 | 2 | 2 | 11 | Huge impact but complex & expensive |
| Cluster 6: Client-Side | 2 | 4 | 4 | 5 | 15 | Low impact (doesn't fix root cause), easy to build |
| Hybrid A: Smart Cache + Progressive | 5 | 4 | 4 | 4 | 17 | Best of both worlds, some complexity |
| Hybrid B: Pre-compute + Edge | 5 | 3 | 3 | 3 | 14 | Very fast responses, needs infra investment |
| Hybrid C: Reduce + Cache | 5 | 5 | 5 | 5 | 20 | Highest score - low-hanging fruit + caching |

**Top 5 Concepts**:
1. **Hybrid C: Reduce + Cache** (Score: 20) - Wins on all dimensions. Remove unused features, then cache what remains. Quick to ship, low cost.
2. **Cluster 1: Caching** (Score: 18) - Proven, easy to implement. Redis with 15-min TTL gets 80% of the win with minimal effort.
3. **Hybrid A: Smart Cache + Progressive** (Score: 17) - Better UX than pure caching. Users see critical data instantly (< 200ms) while rest loads.
4. **Cluster 3: Progressive Load** (Score: 16) - Good UX band-aid while we work on caching. Can ship in parallel.
5. **Cluster 2: Pre-computation** (Score: 14) - More engineering effort but unlocks future features (real-time updates via websocket from pre-computed data).

**Trade-off Analysis**:
- **Hybrid C** is a sweet spot: high impact, high feasibility, fast to ship, low cost. The risk is product decision (which widgets to remove) might face pushback.
- **Cluster 1 (Caching)** is the safest bet: no product changes, pure engineering solution. Downside: if cache is stale during high-activity periods, users might see outdated data.
- **Hybrid A** has best UX but requires both caching infrastructure AND frontend changes (progressive loading). More moving parts = more risk.

---

### Phase 5: Top Ideas (Refined & Decompose-Ready)

### Idea #1: Reduce & Cache Strategy

**Pitch**: Remove unused dashboard widgets (50% of queries), then cache the remaining critical queries in Redis with 15-minute TTL.

**Approach**: First, analyze dashboard analytics to identify which widgets users actually interact with. Remove widgets that <10% of users ever click. Then, add Redis caching layer for the remaining 3-5 expensive aggregate queries. API checks Redis first; on miss, queries DB and backfills cache.

**Why This Could Work**:
- **Data-driven reduction**: Analytics show 5 of 8 widgets are never clicked after initial dashboard load. Removing them eliminates 62% of query time.
- **Caching is proven**: Redis can serve cached dashboard data in 10-20ms vs 3-5 seconds from Postgres. 15-minute TTL is acceptable per user research.
- **Low risk**: Changes are additive (Redis layer) + subtractive (remove features). No complex rewrites. Can roll back easily.

**Key Assumptions**:
- Users actually don't need the widgets we plan to remove (validate with analytics + user interviews)
- 15-minute staleness is acceptable (validate with product team + user research)
- Redis won't become a bottleneck (capacity plan: 100K users × 50KB/dashboard = 5GB, well within single Redis instance)

**Biggest Risk**: Product team pushes back on removing widgets. Mitigation: Start with A/B test - hide widgets for 10% of users, measure engagement drop (hypothesis: minimal).

**First Step**: Run dashboard analytics query: `SELECT widget_id, COUNT(DISTINCT user_id) as users, AVG(click_count) as avg_clicks FROM widget_interactions GROUP BY widget_id ORDER BY users ASC`. Identify bottom 3 widgets for removal.

**Evaluation Scores**: Impact: 5, Feasibility: 5, Speed: 5, Cost: 5

---

### Idea #2: Redis Caching with Smart Invalidation

**Pitch**: Cache entire dashboard response in Redis with intelligent cache invalidation when underlying data changes.

**Approach**: Add Redis as caching layer. On dashboard request: (1) Check Redis for cached response, (2) If hit, return immediately (10-20ms), (3) If miss, query DB, serialize, store in Redis with 15-min TTL, return to user. Add cache invalidation hooks: when users create/update entities that appear on dashboard, invalidate that user's cache entry.

**Why This Could Work**:
- **Massive speedup**: 10-20ms cached response vs 3-5 sec DB queries = 150-250x improvement
- **Acceptable staleness**: Most dashboard data (project stats, user counts) doesn't change every minute. 15-minute TTL balances freshness and speed.
- **Smart invalidation**: Users who make changes get fresh data immediately (cache invalidated on write). Passive viewers get fast cached data.

**Key Assumptions**:
- Redis infrastructure is available or easy to provision (validate: check if team already uses Redis for sessions or rate limiting)
- 15-minute staleness is acceptable for majority use case (validate with product team)
- Cache invalidation logic won't become complex/buggy (risk: invalidation triggers in 20+ places across codebase)

**Biggest Risk**: Cache invalidation is hard. We might miss invalidation triggers, causing users to see stale data after making changes. Mitigation: Start with simple TTL-only approach (no smart invalidation), add invalidation incrementally for high-value cases.

**First Step**: Add Redis instance (or use existing if available). Write simple caching middleware: `cache_key = f"dashboard:{user_id}", cached = redis.get(cache_key), if cached: return cached, else: data = fetch_dashboard(), redis.setex(cache_key, 900, data), return data`.

**Evaluation Scores**: Impact: 4, Feasibility: 5, Speed: 5, Cost: 4

---

### Idea #3: Progressive Loading with Critical-First Strategy

**Pitch**: Return critical dashboard widgets instantly from cache, then stream remaining widgets asynchronously as they're computed.

**Approach**: Split dashboard into "critical" (above-fold: user stats, recent activity) and "secondary" (below-fold: charts, analytics). On request: (1) Return critical widgets from Redis cache immediately (< 200ms), (2) Return HTTP 200 with partial payload, (3) Kick off background jobs to compute secondary widgets, (4) Use Server-Sent Events (SSE) or polling to stream secondary widgets to frontend as they're ready, (5) Frontend progressively renders widgets as data arrives.

**Why This Could Work**:
- **Perceived performance**: Users see useful data in 200ms (critical widgets), even if full dashboard takes 2-3 seconds. Feels much faster than 3-5 second blank screen.
- **Prioritization**: Critical data (user name, primary metrics) is always fast. Nice-to-have data (charts, analytics) loads progressively.
- **Graceful degradation**: If secondary widget queries fail/timeout, critical data is already shown. Better UX than all-or-nothing.

**Key Assumptions**:
- Users will tolerate progressive loading (staggered content appearing). Modern web users are accustomed to this (Facebook, Twitter).
- We can identify which widgets are "critical" vs "secondary" (validate with user research: eye tracking, analytics on scroll depth)
- Frontend can handle progressive rendering without janky layout shifts (need skeleton loaders, reserved space)

**Biggest Risk**: Implementation complexity. Requires coordination between backend (SSE/polling), frontend (progressive rendering), and caching layer. More moving parts than simple caching solution. Also, SSE might not work behind some corporate proxies/firewalls.

**First Step**: Prototype with 2-tier loading (no SSE yet): Return cached critical widgets immediately with `{status: "partial", data: {...}}`, include `secondary_data_url` for frontend to fetch via second API call. Test user perception vs current all-at-once loading.

**Evaluation Scores**: Impact: 5, Feasibility: 4, Speed: 4, Cost: 4

---

### Idea #4: Background Pre-Computation with Event-Driven Updates

**Pitch**: Background job pre-computes dashboard data every 5 minutes for all active users, stores in Redis. API serves pre-computed data instantly. When user makes changes, trigger immediate re-computation.

**Approach**: Set up cron job (or scheduled background worker) that runs every 5 minutes: (1) Identify "active" users (logged in within last 24 hours), (2) For each active user, compute dashboard data, (3) Store in Redis with key `dashboard:user_id`, (4) API endpoint simply reads from Redis (10-20ms response). Add event-driven triggers: when user creates/updates entities, enqueue high-priority job to recompute their dashboard immediately.

**Why This Could Work**:
- **Consistently fast**: Every request is a Redis read (10-20ms), no DB queries. Eliminates variability (no more 3-5 second responses).
- **Scalable**: Pre-computation runs during low-traffic periods. API servers don't do heavy lifting, they just serve cached data.
- **Fresh when it matters**: Event-driven re-computation means users see changes immediately after actions (better than 15-min stale cache).

**Key Assumptions**:
- We can identify "active" users accurately (e.g., session in last 24h). We don't want to pre-compute for all users (wasteful).
- Background job can finish pre-computing all active users within 5 minutes (capacity plan: 10K active users × 200ms/dashboard = 2000 seconds = 33 minutes. Need to parallelize or reduce window).
- Event-driven triggers won't cause thundering herd (e.g., bulk import triggers 1000 re-computations). Need rate limiting.

**Biggest Risk**: Background job infrastructure. If we don't already have a robust job queue (Sidekiq, Celery), this adds operational complexity. Also, need to handle job failures gracefully (retry logic, dead letter queue).

**First Step**: Measure: How many "active" users do we have? Run query: `SELECT COUNT(DISTINCT user_id) FROM sessions WHERE last_activity > NOW() - INTERVAL '24 hours'`. If < 1000, this approach is very feasible. If > 10K, need to optimize (only pre-compute for users likely to visit dashboard based on usage patterns).

**Evaluation Scores**: Impact: 5, Feasibility: 3, Speed: 3, Cost: 3

---

### Idea #5: Data Reduction via Widget Removal & Lazy Loading

**Pitch**: Remove widgets that users rarely interact with, and lazy-load remaining widgets only when user clicks to expand them.

**Approach**: Analyze dashboard widget engagement (click rate, expansion rate, time spent viewing). Remove bottom 50% of widgets entirely (or move to "Advanced" tab that <5% of users visit). For remaining widgets, show collapsed/summary view by default (e.g., chart title + one-line summary). Only fetch full widget data when user clicks "Expand". This reduces initial payload from 8 queries to 2-3 queries.

**Why This Could Work**:
- **Immediate impact**: Removing 5 of 8 queries reduces response time from 3-5 sec to ~1 sec with no other changes. Quick win.
- **Better UX**: Paradox of choice - fewer widgets means cleaner, more focused dashboard. Users can find what they need faster.
- **Lazy loading**: Users who need advanced widgets can still access them, but we don't penalize the 90% who don't.

**Key Assumptions**:
- Users won't revolt over removed widgets (validate: the widgets we remove are actually low-engagement, not just low-click-but-high-value)
- Lazy loading is acceptable UX for secondary widgets (vs everything visible upfront)
- Product team is willing to make opinionated decisions about what's "core" vs "advanced"

**Biggest Risk**: Product/design pushback. Removing features is politically hard, even if data supports it. Mitigation: Frame as A/B test ("let's test a focused dashboard for 10% of users and measure engagement"), not permanent removal.

**First Step**: Export dashboard widget analytics: `SELECT widget_id, SUM(views) as total_views, SUM(clicks) as total_clicks, AVG(time_spent_sec) as avg_time FROM widget_analytics WHERE created_at > NOW() - INTERVAL '30 days' GROUP BY widget_id ORDER BY total_clicks ASC`. Present to product team with recommendation to remove bottom 3.

**Evaluation Scores**: Impact: 4, Feasibility: 5, Speed: 5, Cost: 5

---

## Next Steps

You now have 5 refined ideas ready for deeper exploration:

1. **Best quick win**: Idea #1 (Reduce & Cache) or Idea #5 (Data Reduction) - both are low-effort, high-impact
2. **Decompose for implementation**: "Decompose Idea #2 (Redis Caching)" to get Layer 1 (simple story) → Layer 2 (architecture) → Layer 3 (implementation blueprint)
3. **Prototype & test**: Build a quick prototype of Idea #3 (Progressive Loading) to test user perception
4. **Validate assumptions**: Survey users about 15-minute staleness tolerance (critical for caching approaches)
5. **Hybrid approach**: Combine Idea #5 (remove widgets) + Idea #2 (cache remaining) for maximum impact

**Recommended Path Forward**:
- **Week 1**: Implement Idea #5 (remove unused widgets) - pure product decision, no engineering
- **Week 2**: Implement Idea #2 (Redis caching) on reduced widget set
- **Result**: 10-20ms response time (from 3-5 sec), minimal engineering effort, data-driven product decisions

---

## Example 2: Product Feature - Improving Mobile App Retention

**User Request**: "Our mobile app has 40% churn in the first week. Brainstorm ways to improve retention."

---

### Phase 1: Problem Framing

**Problem Restatements**:
- **How Might We**: How might we give users a reason to come back to the app every day for the first week?
- **Pain to Eliminate**: Users download the app, try it once, find no immediate value, and forget about it
- **Outcome to Achieve**: 70%+ of users who install the app are still active after 7 days
- **Constraint to Remove**: The constraint is that we have no re-engagement mechanism (notifications, emails) in the first week

**Success Criteria**:
- **Must-haves**:
  - Day 7 retention improves from 60% to 70%+ (10 percentage point lift)
  - Solutions work for both iOS and Android
  - No aggressive/spammy tactics (preserve brand reputation)
- **Nice-to-haves**:
  - Also improves Day 30 retention (long-term stickiness)
  - Low development effort (ship within 1 sprint)
  - Works organically (not reliant on paid re-engagement)
- **Evaluation dimensions**: Impact (retention lift), Feasibility (dev effort), Speed (time to ship), Cost (development + ongoing)

**Assumptions to Challenge**:
- We assume users understand the app's value after first session
- We assume users remember to come back without prompting
- We assume the first session experience is good enough
- We assume push notifications are the only re-engagement tool
- We assume all users have similar needs/goals

---

### Phase 2: Divergent Exploration

#### SCAMPER Ideas (7 ideas)

1. **Substitute**: Replace generic onboarding with personalized goal-setting. Ask "What do you want to achieve this week?" and show progress toward that goal.
2. **Combine**: Combine app usage with calendar integration. Auto-schedule daily 5-minute sessions in user's calendar.
3. **Adapt**: Duolingo's streak mechanic - show "3 day streak!" and a calendar view of daily activity. Loss aversion kicks in.
4. **Modify**: Modify push notifications to be contextual (time of day, user behavior) instead of generic "Come back!" messages.
5. **Put to other use**: Repurpose email (which users check daily) for re-engagement instead of relying on push notifications (which users ignore).
6. **Eliminate**: Eliminate complex onboarding. Just show ONE core feature in first session, nothing else. Reduce overwhelm.
7. **Reverse**: Instead of pushing users to come back, pull them in by creating FOMO (show what other users are achieving, real-time activity feed).

#### First Principles (5 ideas)

8. Strip to core: The fundamental problem is "users don't experience value in first session." Fix that first. Run user interviews to find "aha moment" and optimize for it.
9. Challenge assumption: We assume retention = daily use. What if we optimize for weekly use instead? Less pressure on users, more sustainable.
10. Build from scratch: If we had no app history, we'd build a "daily challenge" feature that gives users a specific, small task each day. Can we add this?
11. What if we could: What if we could guarantee users achieve a small win in first session? Design onboarding around guaranteed success (e.g., "Complete 3 easy tasks → earn badge").
12. Rethink the metric: "Retention" is wrong metric. We should measure "value delivered in first week." If users get value, retention follows.

#### Analogical Thinking (6 ideas)

13. **From gyms**: Gyms offer free personal training session for new members. Offer free 1:1 onboarding call or live chat support in first week.
14. **From social media**: Instagram shows "Suggested for you" to new users. Show "People like you are achieving X" to inspire.
15. **From games**: Games have daily login rewards (escalating value). Day 1 = 10 coins, Day 7 = 100 coins. Make it valuable to log in daily.
16. **From newspapers**: Newspapers send daily digest emails. Send daily "Here's what's new for you" email with personalized content.
17. **From fitness apps**: Strava has clubs/communities. Create beginner cohorts - "You and 50 others started this week, here's their progress."
18. **From habit apps**: Habit trackers show calendar heat map. Visualize user's activity pattern - seeing blank days motivates filling them in.

#### Constraint Removal (6 ideas)

19. **Unlimited budget**: Hire onboarding specialists to personally call every new user, understand their goals, and set them up for success. (Then scale with AI chatbot.)
20. **Unlimited time**: Rewrite onboarding from scratch based on 100+ user interviews. Build adaptive onboarding that changes based on user persona.
21. **Perfect data**: If we knew exactly which feature each user needs most, we'd show only that feature first. Use ML to predict and personalize onboarding.
22. **No regulations**: If we could send unlimited push notifications, we'd send 5/day in first week with different value props. (Then scale back based on what works.)
23. **10% version of #19**: Can't call everyone, but can we send personalized video message to new users? "Hi [Name], welcome! Here's how to get started..."
24. **Adjacent win from #21**: Don't have ML yet, but we can segment users by signup source (ads, organic, referral) and customize onboarding per segment.

#### Inversion & Provocation (6 ideas)

25. **Make it worse, then invert**: Make it worse by adding more features → Invert: Remove all features except one killer feature. If users love that, they'll come back.
26. **Stupidest idea**: Pay users $1/day to log in for first week. Insight: We could offer in-app currency/credits instead (cheaper, still motivating).
27. **What competitors won't do**: Competitors send generic notifications. We send personal, human messages ("Hey [Name], I noticed you tried X - here's how to get more value").
28. **Opposite of obvious**: Instead of trying to get users to open app daily, optimize for weekly deep session (1 hour on Sunday). Less frequent but more valuable.
29. **Provocative reframe**: The problem isn't retention. It's that we're attracting wrong users (people who don't have the problem we solve). Fix acquisition, not retention.
30. **Wildcard**: What if we made the app multiplayer? Invite a friend, compete/collaborate. Social pressure drives return visits.

**Total**: 30 raw ideas

---

### Phase 3: Clustering & Combination

#### Cluster 1: Personalized Onboarding
- **Ideas**: #1 (goal-setting), #8 (aha moment), #11 (guaranteed win), #20 (adaptive onboarding), #24 (segment by source)
- **Core approach**: Customize first session to user's needs/goals to maximize value delivered

#### Cluster 2: Re-Engagement Mechanics
- **Ideas**: #4 (contextual notifications), #5 (email re-engagement), #7 (FOMO/activity feed), #16 (daily digest)
- **Core approach**: Remind users to come back with timely, relevant prompts

#### Cluster 3: Habit Formation
- **Ideas**: #3 (streak mechanic), #10 (daily challenge), #15 (login rewards), #18 (calendar heat map)
- **Core approach**: Use behavioral psychology to build daily habit

#### Cluster 4: Social/Community
- **Ideas**: #14 (people like you), #17 (beginner cohorts), #30 (multiplayer)
- **Core approach**: Leverage social dynamics (inspiration, accountability, competition)

#### Cluster 5: Simplification
- **Ideas**: #6 (eliminate complexity), #25 (one killer feature), #28 (optimize for weekly vs daily)
- **Core approach**: Reduce cognitive load by showing less, not more

#### Cluster 6: Human Touch
- **Ideas**: #13 (1:1 onboarding), #19 (personal calls), #23 (personalized video), #27 (human messages)
- **Core approach**: Make users feel personally supported (doesn't scale, but high impact)

#### Hybrid Combinations

- **Hybrid A: Personalized Goal + Streak**: Combine #1 (goal-setting) + #3 (streak mechanic). In onboarding, ask "What's your goal this week?" (e.g., "Read 3 articles"). Track progress toward goal and show streak calendar. Personal + gamified.

- **Hybrid B: Simplified Onboarding + Daily Challenge**: Combine #6 (show one feature) + #10 (daily challenge). First session: Show ONLY core feature, ensure user completes one task successfully. Days 2-7: Send daily challenge via notification/email to bring them back.

- **Hybrid C: Social Cohorts + Habit Tracking**: Combine #17 (beginner cohorts) + #18 (calendar heat map). Group new users into weekly cohorts (public or anonymous), show cohort's collective progress (heat map of activity). Social accountability + visualization.

#### Standout Outliers

- **Outlier #26 (Pay to engage)**: Paying users directly is extreme, but offering in-app credits/rewards for first-week logins is feasible and proven (mobile games do this).

- **Outlier #29 (Fix acquisition, not retention)**: Provocative but possibly true. If we're attracting users who don't need our solution, no amount of retention tactics will help. Worth investigating signup source quality.

---

### Phase 4: Convergent Evaluation

| Concept | Impact | Feasibility | Speed | Cost | Total | Notes |
|---------|--------|-------------|-------|------|-------|-------|
| Cluster 1: Personalized Onboarding | 5 | 3 | 3 | 4 | 15 | High impact but requires research + dev effort |
| Cluster 2: Re-Engagement Mechanics | 4 | 5 | 5 | 5 | 19 | Proven to work, easy to implement, low cost |
| Cluster 3: Habit Formation | 5 | 4 | 4 | 5 | 18 | Strong behavioral psychology, moderate dev effort |
| Cluster 4: Social/Community | 4 | 3 | 3 | 4 | 14 | High engagement but requires critical mass of users |
| Cluster 5: Simplification | 5 | 5 | 5 | 5 | 20 | Easiest to implement (remove, don't add), highest impact |
| Cluster 6: Human Touch | 3 | 2 | 2 | 2 | 9 | Doesn't scale, expensive, slow to implement |
| Hybrid A: Goal + Streak | 5 | 4 | 4 | 4 | 17 | Strong combination, proven mechanics |
| Hybrid B: Simplified + Daily Challenge | 5 | 5 | 5 | 5 | 20 | Best of both worlds - simple + engaging |
| Hybrid C: Social + Habit Tracking | 4 | 3 | 3 | 4 | 14 | Interesting but needs community features |

**Top 5 Concepts**:
1. **Cluster 5: Simplification** (Score: 20) - Remove complexity from first session. Show one core feature, ensure user succeeds. Paradoxically, less is more for retention.
2. **Hybrid B: Simplified + Daily Challenge** (Score: 20) - Combine simplification with habit-building. Clean first session, then daily nudges to return.
3. **Cluster 2: Re-Engagement Mechanics** (Score: 19) - Low-hanging fruit. Better notifications, email reminders. Easy to ship, proven to work.
4. **Cluster 3: Habit Formation** (Score: 18) - Streaks, daily challenges, visual progress. Taps into behavioral psychology effectively.
5. **Hybrid A: Goal + Streak** (Score: 17) - Personal goal-setting + gamification. High engagement, requires moderate dev effort.

**Trade-off Analysis**:
- **Cluster 5 (Simplification)** is the safest bet: just remove features from onboarding. High impact, zero cost, fast to ship. Risk: Might hide features users actually want (mitigate with user research).
- **Hybrid B** is the best balanced approach: simplify first, then engage with challenges. Requires both UX changes and notification system, but not complex.
- **Cluster 2 (Re-Engagement)** is tactical: won't fix a broken onboarding, but will remind users to come back. Band-aid solution, but effective.

---

### Phase 5: Top Ideas (Refined & Decompose-Ready)

(Abbreviated - similar structure to Example 1)

### Idea #1: Radical Simplification of First Session

**Pitch**: Show only ONE core feature in first session, hide everything else. Ensure user completes one successful action before seeing additional features.

**Approach**: Redesign onboarding to focus on single "aha moment" (identified through user research). Example: If core value is "find relevant content," first session shows personalized feed + one-click save. That's it. After user saves 3 items, unlock other features (search, categories, settings). Progressive disclosure based on success, not time.

**Why This Could Work**:
- Users are overwhelmed by full feature set in first session. Simplifying increases completion rate of core action (which correlates with retention).
- Proven by consumer apps (Instagram, TikTok): new users see feed immediately, not settings/profile/explore tabs.
- Easy to implement: hide UI elements, add progressive disclosure logic.

**Key Assumptions**:
- We've correctly identified the "one core feature" (validate with user interviews + analytics)
- Users won't churn because features are hidden (mitigate: show "Unlock more features" prompt after completing core action)

**Biggest Risk**: Product team resistance ("but we worked hard on those features!"). Mitigation: A/B test, show data on retention lift.

**First Step**: Run user session recordings for first-time users. Identify drop-off points, time to first successful action, feature usage. Hypothesis: users who complete core action in first 2 minutes have 2x retention.

**Evaluation Scores**: Impact: 5, Feasibility: 5, Speed: 5, Cost: 5

---

### Idea #2: Daily Challenge System with Streaks

**Pitch**: Send users a simple, personalized daily challenge via notification/email. Track completion streaks with visual calendar to build habit.

**Approach**: After first session, system generates a daily challenge based on user's behavior (e.g., "Read one article today" or "Save 3 items to your list"). Send push notification at optimal time (based on when user previously opened app). When user completes challenge, show streak counter and calendar heat map ("5 day streak! 🔥"). Escalating rewards: Day 7 = unlock premium feature for 1 week.

**Why This Could Work**:
- Duolingo proved this works: streaks drive 2x engagement vs no-streak control group.
- Daily challenges give users a clear, low-friction reason to return.
- Calendar visualization leverages loss aversion (don't want to break the streak).

**Key Assumptions**:
- Users will engage with push notifications (validate: current open rate, permission rate)
- Daily challenges are achievable in 2-5 minutes (validate: time user behavior data)
- Streak mechanic is motivating for target demographic (validate: survey users on gamification preferences)

**Biggest Risk**: Notification fatigue. Users disable notifications, system fails. Mitigation: Make notifications valuable (personalized challenges, not generic "Come back"), allow users to set preferred time.

**First Step**: Prototype challenge system for 10% of users. Use simple rule-based challenges (no ML): "Complete [last action user did] again today." Measure: notification open rate, challenge completion rate, Day 7 retention vs control.

**Evaluation Scores**: Impact: 5, Feasibility: 4, Speed: 4, Cost: 4

---

(Additional ideas #3-#5 would follow similar structure)

---

## Next Steps

1. **Decompose Idea #1**: "Decompose the Radical Simplification approach" to get Layer 1 (simple story with analogy) → Layer 2 (UX flow, feature gating logic) → Layer 3 (implementation details, A/B test plan)
2. **Prototype Idea #2**: Build MVP of daily challenge system in 1 sprint, test with 10% of new users
3. **Combine ideas**: Idea #1 (simplify first session) + Idea #2 (daily challenges) = comprehensive first-week experience

