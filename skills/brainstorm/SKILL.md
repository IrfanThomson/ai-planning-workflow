---
name: brainstorm
description: Collaborative brainstorming through iterative exploration. Gathers context, researches existing solutions, generates diverse ideas with pros/cons, then refines based on user direction until reaching a solution ready for decomposition.
model: claude-sonnet-4-20250514
---

# Brainstorm: Collaborative Solution Exploration

This skill facilitates iterative brainstorming through conversation. It explores the solution space with diverse ideas, provides pros/cons, and refines based on your direction until you find an approach worth building.

## Workflow Context

**This skill can be used:**
- **As part of the planning pipeline**: `/brainstorm` → `/breakdown` → `/spec` → `/test-reqs` → `/decompose`
- **Standalone**: When you need to explore options for any problem

**When in the pipeline**: Output feeds into `/breakdown` for progressive understanding in layers.

## Execution Rhythm: One Phase Per Turn

**IMPORTANT**: Do NOT rush through multiple phases in one response. Each phase should be its own turn:

1. **Phase 1 (Context Gathering)**: Ask questions → STOP → Wait for answers
2. **Phase 2 (Research)**: Present research → STOP → Get feedback
3. **Phase 3 (Initial Ideas)**: Generate 5-7 ideas → STOP → Get direction
4. **Phase 4 (Iteration)**: Refine based on feedback → STOP → Check if more iteration needed
5. **Phase 5 (Handoff)**: Summarize chosen approach → STOP → Confirm next step

This prevents overwhelming the user and allows course-correction at each stage.

## Core Principle

**User-Directed Iteration**: Generate diverse ideas initially, then let the user steer ("dig into these 3", "focus on cheaper options", "combine these"). The conversation continues until something clicks.

## The Process

### Phase 1: Context Gathering
**Goal**: Understand what you're solving and what constraints exist

Ask about:
- **The problem/goal**: What are you trying to achieve?
- **Constraints**: Time, budget, technical limitations, team size
- **What's been tried**: Previous attempts, why they didn't work
- **Success criteria**: What does a good solution look like?
- **Existing solutions**: Do you know of any existing approaches or tools?

**Output**: Clear problem understanding + constraints + starting points

---

### Phase 2: Research & Landscape
**Goal**: Ground the brainstorm in reality - what exists, what works, what doesn't

Research:
- **Existing solutions**: Tools, services, frameworks that solve this or similar problems
- **Best practices**: What do successful companies/projects do?
- **Analogous problems**: How do other domains handle similar challenges?
- **Known failures**: What's been tried and failed? Why?

**Output**: 3-5 existing approaches with brief pros/cons

---

### Phase 3: Initial Exploration
**Goal**: Cover the solution space with diverse ideas

Generate 5-7 ideas that explore different directions:
- Adapt existing solutions
- Build from scratch with different technical approaches
- Change the process/workflow instead of building
- Hybrid approaches
- Contrarian/unconventional ideas

For each idea, provide:
- **Brief description** (2-3 sentences)
- **Pros**: Why this could work
- **Cons**: What the trade-offs are

**Output**: 5-7 diverse ideas with pros/cons

---

### Phase 4: User-Directed Iteration
**Goal**: Go deeper based on what interests you

**You steer the direction**:
- "Dig into ideas #2 and #5"
- "Focus more on low-cost solutions"
- "What if we combined these two approaches?"
- "I don't like any of these, give me more contrarian ideas"
- "Explore the technical implementation of #3"
- **"Brainstorm idea #1 as a nested session"** (see Nested Brainstorming below)

I'll generate new ideas/variations based on your direction.

**This phase repeats** until you say something like:
- "Yes, idea #4 feels right"
- "I want to go with this approach"
- "Let's decompose this one"

**Output**: Refined ideas based on your steering, with deeper analysis

---

### Phase 5: Handoff
**Goal**: Package the chosen idea for decomposition

When you've picked an approach, I'll:
- Summarize the chosen solution
- Clarify any remaining ambiguities
- Ask if you want to decompose it

Then you can say: **"Decompose this"** to break it down into Layer 1 (simple story) → Layer 2 (architecture) → Layer 3 (implementation blueprint)

---

## Nested Brainstorming (Russian Nesting Dolls)

**Concept**: Sometimes during brainstorming, a single idea deserves its own full brainstorming process. You can "dive into" a specific idea, run through all 5 phases again at a more detailed level, then "pop back" to the parent brainstorm.

### When to Use Nested Brainstorming

- An idea is promising but too vague/broad to decompose directly
- Multiple implementation approaches exist for one idea
- You want to explore trade-offs within a single approach before committing
- The idea touches on a complex sub-problem worth exploring independently

### How It Works

**Starting a Nested Session:**
User says: "Brainstorm idea #1" or "Let's dive deeper into the Success Story Mining idea"

I will:
1. **Mark the parent context**: "Pausing main brainstorm (Windsurf Innovations). Diving into nested brainstorm: Success Story Mining System"
2. **Restart Phase 1**: Gather context specific to this sub-idea
3. **Run through all 5 phases**: Research, explore, iterate, handoff
4. **Track the nesting level**: Show breadcrumb like "Parent: Windsurf Innovations → Current: Success Story Mining"

**Returning to Parent:**
User says: "Pop back" or "Return to parent brainstorm" or "Back to Windsurf ideas"

I will:
1. **Summarize nested outcome**: "From nested brainstorm we identified: AI-Powered Story Detector as preferred approach"
2. **Reintegrate insights**: "This becomes a refined version of Idea #1 in the parent brainstorm"
3. **Resume parent session**: "Back to Windsurf Innovations brainstorm. You were exploring ideas #1, #3, #7. Want to dive into #3 next?"

### Nesting Depth

- **Level 0**: Top-level brainstorm (e.g., "Windsurf Innovations")
- **Level 1**: Nested brainstorm (e.g., "Success Story Mining variations")
- **Level 2**: Double-nested brainstorm (e.g., "Interview Bot conversation design")

Recommended max depth: **Level 2**. Beyond that, consider decomposing instead.

### Visual Breadcrumbs

I'll show the nesting context like this:

```
🔹 BRAINSTORM CONTEXT
Level 0: Windsurf Innovations (7 ideas generated)
  └─ Level 1: Success Story Mining System (currently in Phase 3 - exploring 5 variations)
```

When you pop back:

```
🔹 BRAINSTORM CONTEXT
Level 0: Windsurf Innovations (currently in Phase 4 - iterating on ideas #1, #3, #7)
  └─ Level 1: Success Story Mining System [COMPLETED - chose AI-Powered Story Detector]
```

### Example Flow

```
User: "Brainstorm innovations on Windsurf"
[Phase 1-3: Generate 7 ideas including "Success Story Mining System"]

User: "Brainstorm idea #1"
Assistant:
🔹 NESTED BRAINSTORM STARTED
Parent: Windsurf Innovations
Current: Success Story Mining System

Let me gather context for this sub-idea:
- What's the primary goal: prove ROI, inspire adoption, or capture knowledge?
- Who are the stakeholders: developers, managers, executives?
[... full brainstorm process for this idea ...]

User: "I like the AI-Powered Story Detector variation"
Assistant: Great! This is the refined version of Idea #1 from parent brainstorm.

User: "Pop back to main brainstorm"
Assistant:
🔹 RETURNED TO PARENT BRAINSTORM
Level 0: Windsurf Innovations

Summary from nested session:
- Chose: AI-Powered Story Detector (monitors patterns, interviews devs, generates stories)
- Key insight: Balances automation with human context

You were exploring ideas #1, #3, #7.
- ✓ Idea #1 refined via nested brainstorm
- Want to dive into #3 (Rules Marketplace) next?
```

---

## Execution Guidelines

### Context Gathering
- Ask open-ended questions, don't assume
- Clarify constraints (is budget tight? is speed critical?)
- Understand "why now" (what triggered this need?)

### Research Phase
- Use web search to find existing solutions if needed
- Don't just list tools - explain what approach they take
- Note what's missing from existing solutions (gaps we could fill)

### Idea Generation
- **Diverse, not exhaustive**: Cover different types of approaches, not every possible variation
- **Concrete, not vague**: "Use Redis caching" not "make it faster"
- **Honest pros/cons**: Every idea has trade-offs, don't sell
- **Build on context**: Reference constraints and criteria from Phase 1

### Iteration
- Follow the user's direction explicitly
- If they ask to dig into 3 ideas, focus only on those 3
- If they ask to combine ideas, show how that hybrid would work
- If they push back, explore why and offer alternatives

### Knowing When You're Done
- User explicitly picks an idea
- User says "let's go with this" or similar
- User asks to decompose or plan implementation
- Don't force convergence - some brainstorms need multiple sessions

---

## Example Flow

```
User: "Brainstorm ways to speed up our slow dashboard API"
Assistant (Phase 1): Let me understand the context first.
- How slow is it currently? What's the target response time?
- Are there budget or timeline constraints?
- Have you tried anything already?
- Do you know of any existing solutions or approaches?

User: "It takes 3-5 seconds, want under 1 second. No budget issues. Haven't tried anything yet."

Assistant (Phase 2): Let me research existing approaches:

**Existing Solutions**:
1. **Redis caching** - Store computed results, serve from memory
   - Pros: Fast (sub-50ms), proven, well-supported
   - Cons: Stale data, cache invalidation complexity
2. **Database indexing** - Optimize slow queries
   - Pros: No architecture change, targets root cause
   - Cons: May not be enough if queries are complex
3. **GraphQL DataLoader** - Batch and cache queries
   - Pros: Reduces N+1 queries, built-in caching
   - Cons: Requires GraphQL migration

Assistant (Phase 3): Here are 6 diverse initial ideas:

**Idea 1: Add Redis caching layer**
Cache entire dashboard response for 5-10 minutes.
- Pros: Quick win, massive speedup
- Cons: Stale data, cache invalidation headaches

**Idea 2: Pre-compute dashboard data via background job**
Background worker computes dashboard every 5 min, stores in Redis. API just reads.
- Pros: Consistently fast, no query time
- Cons: Requires job infrastructure, still some staleness

**Idea 3: Progressive loading**
Return critical data immediately, stream remaining data.
- Pros: Perceived performance boost
- Cons: More frontend complexity, doesn't solve root problem

**Idea 4: Remove expensive queries**
Analyze which queries are slowest, remove or simplify those dashboard widgets.
- Pros: Zero cost, might improve UX (less clutter)
- Cons: Product decision, might lose features users want

**Idea 5: Read replica + materialized views**
Set up read replica, create materialized views for dashboard queries.
- Pros: Separates read/write load, very fast reads
- Cons: Infra complexity, cost increase

**Idea 6: Rewrite with Elasticsearch**
Index data in Elasticsearch, use aggregations for dashboard.
- Pros: Built for analytics queries, extremely fast
- Cons: Major architectural change, operational complexity

Where do you want to dig deeper?

User: "Dig into ideas 1 and 2. Also, what if we combined them?"

Assistant (Phase 4 - Iteration): Let me explore caching and pre-computation:

**Idea 1 Deep Dive: Redis Caching**
Implementation: API checks Redis first (key: `dashboard:user_id`), if miss queries DB and backfills cache with 10-min TTL.

Variations:
- **1a: Simple TTL** - Let data go stale for 10 min, simple but some staleness
- **1b: Smart invalidation** - Invalidate cache when user makes changes, fresh data when it matters
- **1c: Tiered caching** - Cache different widgets with different TTLs based on update frequency

Pros: Easy to implement (1-2 days), proven pattern, can iterate on invalidation strategy
Cons: Cache invalidation is hard, might miss edge cases

**Idea 2 Deep Dive: Background Pre-Computation**
Implementation: Cron job runs every 5 min, computes dashboard for "active users", stores in Redis. API just reads.

Variations:
- **2a: All active users** - Pre-compute for anyone who logged in last 24h
- **2b: Predictive** - Only pre-compute for users likely to request dashboard (based on patterns)
- **2c: On-demand trigger** - Pre-compute on user login, not on schedule

Pros: Consistent performance, separates heavy computation from request path
Cons: Need job infrastructure, might waste computation on users who don't visit

**Combined Idea: Smart Cache + On-Demand Pre-Compute**
Hybrid approach:
1. API checks Redis cache first
2. If miss, return "computing..." and trigger background job
3. Background job computes dashboard, stores in Redis
4. Frontend polls or uses SSE to get result when ready
5. Subsequent requests hit cache (10-min TTL)

Pros: Best of both - fast when cached, graceful degradation on miss, no wasted pre-computation
Cons: Most complex implementation, frontend needs to handle async loading

What do you think?

User: "I like the combined approach. Let's decompose that."

Assistant (Phase 5 - Handoff): Great! The combined approach is:
**Smart Cache + On-Demand Pre-Compute** - API serves from cache when available, triggers background computation on miss, frontend polls for result.

Want me to decompose this into layers?
- Layer 1: Simple story with analogy
- Layer 2: Architecture (API flow, caching strategy, job queue)
- Layer 3: Implementation blueprint (code structure, error handling, monitoring)
```

---

## Integration with Decompose

**Workflow**:
1. **Brainstorm** → Explore solution space until you find an approach
2. **Decompose** → Break it down into Layer 1 (intuition) → Layer 2 (architecture) → Layer 3 (implementation)
3. **Build** → Use Layer 3 as implementation guide

**When to use each**:
- Use **brainstorm** when you need to explore options, aren't sure what to build, or want to validate ideas
- Use **decompose** when you've picked an approach and want to understand it progressively before implementing

---

## Tips for Good Brainstorming

**As the brainstormer:**
- Start with diverse ideas (cover the space)
- Be honest about pros/cons (don't oversell)
- Follow user direction exactly
- Research existing solutions (ground in reality)
- Don't force convergence (it's okay to end without picking)
- **Track nested context clearly** (use breadcrumbs, summarize when popping back)
- **Maintain parent state** (remember where we were when returning)

**As the user:**
- Share constraints upfront (saves time)
- Steer explicitly ("focus on X", "combine these")
- Push back if nothing feels right
- Don't feel pressure to pick something (can brainstorm again later)
- Once something clicks, decompose it to validate the approach
- **Use nested brainstorming** when an idea needs more exploration before committing
- **Pop back strategically** when you've learned enough from the nested session

