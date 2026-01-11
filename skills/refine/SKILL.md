---
name: refine
description: Refines ideas and defines scope through interactive Q&A before technical breakdown. Prevents over-design by establishing clear boundaries, constraints, and priorities. The critical HITL checkpoint that grounds breakdown in actual user intent.
model: claude-sonnet-4-20250514
---

# Refine: Scope Definition Through Interactive Q&A

This skill sits between brainstorm and breakdown to prevent AI from creating overly complex "dream versions" of your idea. It establishes scope, constraints, and priorities through back-and-forth questions, giving breakdown the context it needs to make informed technical decisions.

## Workflow Context

**This skill can be used:**
- **As part of the planning pipeline**: `/brainstorm` → `/refine` → `/breakdown` → `/spec` → `/test-reqs` → `/decompose`
- **Standalone**: When you have an idea and need to define scope before technical planning

**When in the pipeline**: Takes output from `/brainstorm` (chosen approach) and refines it through Q&A to establish clear scope. Output feeds into `/breakdown`, providing context for technical decisions.

## Execution Rhythm: Conversational Iteration

**This skill is CONVERSATIONAL** - it asks questions, you answer, it asks follow-ups until scope is clear.

**Typical flow**:
1. **Initial questions** (5-7 questions about scope, constraints, priorities) → STOP → Wait for answers
2. **Follow-up questions** (based on answers, dig deeper) → STOP → Wait for answers
3. **Clarifications** (resolve ambiguities) → STOP → Confirm understanding
4. **Summary** (refined scope document) → STOP → Get approval
5. **Handoff** (ready for breakdown)

**Don't rush** - this is the critical HITL checkpoint that prevents over-design.

## Core Principle

**AI naturally drifts toward complexity.** Without clear boundaries, breakdown will create comprehensive, feature-rich technical designs that exceed what you actually need. Refinement establishes those boundaries upfront.

**Key insight**: The technical decisions breakdown makes should be **grounded in refinement answers**, not AI assumptions.

---

## The Process

### Phase 1: Understand the Chosen Approach
**Goal**: Review brainstorm output and understand what was picked

**Review**:
- What approach was chosen from brainstorm?
- What problem does it solve?
- What was the rationale for picking this approach?

**Output**: Clear understanding of starting point

---

### Phase 2: Scope & Constraints Questions
**Goal**: Establish boundaries before getting technical

**Ask about**:

**1. Version & Timeline**
- Is this MVP/v1 or full-featured version?
- What's the timeline? (Days, weeks, months)
- What's good enough to ship?

**2. Must-Have vs Nice-to-Have**
- What are the absolute must-haves?
- What features can wait for v2?
- What's the simplest version that delivers value?

**3. Existing vs New**
- What already exists that we're building on?
- What can we use off-the-shelf vs build custom?
- Are there existing patterns/libraries we should follow?

**4. Constraints**
- Tech stack constraints? (Must use X, can't use Y)
- Performance requirements? (Must handle X users, <Y latency)
- Budget constraints? (Free tier only, avoid paid services)
- Team constraints? (Solo developer, team of 3, etc.)

**5. Complexity Budget**
- How complex can this be? (Simple CRUD vs distributed system)
- Is optimization important or can it be naive?
- How much error handling? (Basic vs comprehensive)

**6. User/Usage**
- Who will use this? (Just you, team, public)
- How many users? (1, 100, 10000)
- Usage patterns? (Once a day, real-time, batch processing)

**7. Integration**
- What does this integrate with?
- What depends on this?
- Can this be standalone or must it fit existing architecture?

**Output**: Answers to scope questions

---

### Phase 3: Follow-Up & Clarification
**Goal**: Dig deeper on ambiguous or critical areas

**Based on answers, ask follow-ups like**:

If user said "MVP":
- "What specific features define MVP vs full version?"
- "What's the one feature that must work well vs just work?"

If user said "must be fast":
- "How fast? <100ms, <1s, <5s?"
- "For what operations? All or just specific ones?"

If user said "use existing auth":
- "Which auth system? How does it integrate?"
- "Can we assume it handles X, Y, Z?"

If user said "simple":
- "Simple implementation (less code) or simple UX (more code)?"
- "What complexity are you willing to accept?"

**Output**: Clarified scope with specifics, not vague statements

---

### Phase 4: Priority & Trade-offs
**Goal**: Understand what matters most when trade-offs arise

**Ask**:
- "If you had to choose: feature completeness or simplicity?"
- "If you had to choose: performance or quick implementation?"
- "If you had to choose: flexibility or opinionated design?"
- "What would you regret NOT having in v1?"
- "What can definitely wait until users ask for it?"

**Output**: Clear priorities for breakdown to reference

---

### Phase 5: Refinement Summary
**Goal**: Document refined scope for breakdown to use

**Generate summary**:

```markdown
# Refined Scope: [Feature Name]

## Version & Goals
- **Version**: MVP / v1 / Full-featured
- **Timeline**: [Days/weeks/months]
- **Primary Goal**: [What this must accomplish]

## Must-Have Features
1. [Feature 1 - specific]
2. [Feature 2 - specific]
3. [Feature 3 - specific]

## Explicitly Out of Scope (v2+)
1. [Feature that can wait]
2. [Optimization that's not critical]
3. [Nice-to-have that adds complexity]

## Constraints
- **Tech Stack**: [Must use / Can't use / Prefer]
- **Performance**: [Requirements or "not critical"]
- **Complexity Budget**: Simple / Medium / Complex
- **Timeline**: [Specific deadline if any]

## Integration Context
- **Builds on**: [Existing systems/code]
- **Uses**: [Libraries/services that are given]
- **Integrates with**: [External systems]

## Priorities (when trade-offs arise)
1. [First priority - e.g., Simplicity over completeness]
2. [Second priority - e.g., Speed to ship over optimization]
3. [Third priority - e.g., Maintainability over flexibility]

## Technical Preferences
- [Any specific tech choices user cares about]
- [Patterns to follow or avoid]
- [Libraries/approaches preferred]

## Key Clarifications
- [Important decisions made during refinement]
- [Ambiguities resolved]
- [Assumptions validated]
```

**Present to user**: "Does this capture your intent? Any adjustments?"

**Output**: Approved refinement document

---

### Phase 6: Handoff to Breakdown
**Goal**: Provide context for breakdown Layer 3

**What breakdown gets**:
- Refined scope document (from Phase 5)
- Clear boundaries (what's in/out)
- Priorities for trade-off decisions
- Technical constraints and preferences

**Breakdown should**:
- Use must-haves to drive feature list
- Respect out-of-scope to avoid feature creep
- Reference constraints when making tech choices
- Follow priorities when multiple approaches exist
- Use clarifications to make informed assumptions

**Output**: Context-rich handoff to breakdown

---

## Example Refinement Session

### Input from Brainstorm
"User authentication with email/password and session management"

### Refinement Q&A

**Assistant**: Let me refine the scope for your authentication system. I'll ask some questions:

**Scope Questions**:
1. Is this MVP or full-featured auth?
2. Timeline - days, weeks, or months?
3. Must-haves: Just email/password login, or also signup/logout/session management?
4. Nice-to-haves that can wait: Password reset? Email verification? OAuth? MFA?
5. Existing systems: Starting from scratch or adding to existing app?
6. Tech constraints: Any specific libraries or approaches required?
7. Complexity: Simple (cookie sessions) or complex (JWT, refresh tokens)?

**User**:
- MVP for now, need it working in a week
- Must-haves: Signup, login, logout, sessions that persist
- Can wait: Password reset, email verification, OAuth, MFA
- Adding to existing Express app with PostgreSQL
- Prefer simple cookie sessions, not JWT
- Keep it simple, optimization doesn't matter yet

**Assistant**: Thanks! A few follow-ups:

1. Sessions persist across browser restarts, or just during browser session?
2. How long should sessions last? 24 hours? 7 days?
3. Account lockout after failed attempts, or not critical for MVP?
4. Password strength requirements, or accept any password?
5. Already have user model in DB, or need to create?

**User**:
- Persist across restarts, 7 day expiry
- No lockout, keep it simple
- Basic validation (min 8 chars), no complexity requirements
- Need to create user model

**Assistant**: Got it. One more:

If you had to choose between feature completeness (signup+login+logout+password validation) or shipping faster (just login+logout, add signup later), which matters more?

**User**: Feature completeness. Need signup+login+logout working.

**Assistant**: Perfect. Here's the refined scope:

```markdown
# Refined Scope: User Authentication System

## Version & Goals
- **Version**: MVP
- **Timeline**: 1 week
- **Primary Goal**: Enable users to create accounts and log in to existing Express app

## Must-Have Features
1. User signup with email/password (min 8 chars)
2. User login with email/password
3. User logout
4. Cookie-based sessions (7 day expiry, persist across restarts)
5. User model in PostgreSQL

## Explicitly Out of Scope (v2+)
1. Password reset flow
2. Email verification
3. OAuth (Google, GitHub, etc.)
4. Multi-factor authentication
5. Account lockout after failed attempts
6. Complex password requirements (symbols, numbers, etc.)

## Constraints
- **Tech Stack**: Express.js + PostgreSQL (given)
- **Sessions**: Cookie-based (NOT JWT)
- **Performance**: Not critical for MVP
- **Complexity Budget**: Simple - prioritize working over optimized
- **Timeline**: 1 week

## Integration Context
- **Builds on**: Existing Express app with PostgreSQL
- **Uses**: express-session, bcrypt for password hashing
- **Integrates with**: Existing app routes (will protect with auth middleware)

## Priorities (when trade-offs arise)
1. Simplicity over completeness (v1 ships fast)
2. Working over optimized (make it work, then improve)
3. Security basics over advanced features (hash passwords, secure cookies - but no advanced attack prevention)

## Technical Preferences
- Use cookie sessions (express-session library)
- Use bcrypt for password hashing (NOT plain text)
- PostgreSQL for user storage
- Middleware pattern for protecting routes

## Key Clarifications
- 7 day session expiry is acceptable
- No account lockout needed for MVP
- Password validation is basic (length only)
- Creating new user model in DB (not extending existing)
```

Does this capture your intent?

**User**: Yes, perfect.

**Assistant**: Great! This is ready for `/breakdown`. Breakdown will use this to create a Layer 3 technical blueprint that respects these boundaries - it won't add OAuth, MFA, or other out-of-scope features.

---

## Integration with Workflow

```
/brainstorm → /refine → /breakdown → /spec → /test-reqs → /decompose
                 ↓           ↓
              Scope      Layer 3 Blueprint
              Context    (uses refinement)
```

**How refinement prevents over-design**:

**Without refinement**:
```
Brainstorm: "User authentication"
          ↓
Breakdown: Creates Layer 3 with OAuth, MFA, password reset, email verification,
           JWT refresh tokens, account lockout, password complexity rules,
           role-based permissions, etc.
          ↓
Result: Overwhelmed user, too complex for need
```

**With refinement**:
```
Brainstorm: "User authentication"
          ↓
Refine: MVP, 1 week, email/password only, cookie sessions, no reset/OAuth/MFA
          ↓
Breakdown: Creates Layer 3 with ONLY signup, login, logout, cookie sessions
          ↓
Result: Right-sized solution, matches actual need
```

---

## Question Categories Reference

Use these categories to guide refinement:

### Scope Definition
- MVP vs full version
- Must-have vs nice-to-have
- Simplest version that delivers value
- What can wait for v2

### Timeline & Resources
- How soon is this needed?
- Solo dev or team?
- What's already built vs starting fresh?

### Technical Constraints
- Must use specific tech?
- Can't use certain approaches?
- Existing architecture to fit into?

### Complexity Budget
- How complex can implementation be?
- Optimization important or not?
- Error handling: basic or comprehensive?

### Performance Requirements
- How many users?
- Response time requirements?
- Real-time or batch?

### Integration Points
- What does this connect to?
- What depends on this?
- Standalone or part of larger system?

### User Impact
- Who uses this?
- How often?
- What's the usage pattern?

### Priorities & Trade-offs
- Feature completeness vs simplicity?
- Performance vs quick implementation?
- Flexibility vs opinionated design?

---

## Tips for Effective Refinement

### As the Refiner (AI)
- **Ask specific questions**: "How many users?" not "Usage expectations?"
- **Offer examples**: "Like X or more like Y?" helps users articulate
- **Dig into vague answers**: "Must be fast" → "How fast? <100ms or <5s?"
- **Confirm out-of-scope**: "So X, Y, Z definitely wait for v2?"
- **Prioritize ruthlessly**: "If you can only have 3 features, which 3?"

### As the User
- **Be honest about timeline**: Saying "as soon as possible" doesn't help
- **Know your constraints**: What tech MUST you use? What CAN'T you use?
- **Distinguish must-have from nice-to-have**: Be ruthless about v1 scope
- **Think about simplest version**: What's the 80/20 version?
- **Share technical preferences**: If you care about libraries/patterns, say so

### Common Pitfalls to Avoid
- **Vague scope**: "Make it good" → Ask: "Good means what specifically?"
- **Assuming complexity**: Don't assume user wants the comprehensive version
- **Skipping constraints**: Tech/budget/timeline constraints are critical
- **Not establishing priorities**: Trade-offs WILL happen, establish priorities upfront
- **No out-of-scope**: If everything is in scope, nothing is in scope

---

## When to Use Refinement

**Always use refinement when**:
- Building something from scratch (no existing patterns to follow)
- User said "simple" but idea has potential complexity
- Timeline is tight (days/weeks not months)
- You suspect AI might over-design
- Multiple valid approaches exist with different trade-offs

**Can skip refinement when**:
- Rebuilding something that already exists (clear scope)
- User has very detailed requirements already
- Prototyping/experimentation (scope doesn't matter yet)
- Tiny feature with obvious scope

**Rule of thumb**: If you're unsure whether breakdown will match user intent, refine first.

---

## Output Format

The refinement summary should follow this template:

```markdown
# Refined Scope: [Feature Name]

## Version & Goals
- **Version**: [MVP/v1/Full]
- **Timeline**: [Specific]
- **Primary Goal**: [One sentence]

## Must-Have Features
[Numbered list, specific]

## Explicitly Out of Scope (v2+)
[Numbered list, what's NOT included]

## Constraints
- **Tech Stack**: [Specific choices/requirements]
- **Performance**: [Requirements or "not critical"]
- **Complexity Budget**: [Simple/Medium/Complex]
- **Timeline**: [Deadline if any]

## Integration Context
- **Builds on**: [Existing code/systems]
- **Uses**: [Libraries/services]
- **Integrates with**: [External systems]

## Priorities (when trade-offs arise)
[Ordered list of what matters most]

## Technical Preferences
[User's preferred approaches/libraries]

## Key Clarifications
[Important decisions/assumptions]
```

---

## Usage

**Starting the skill**:
User says: **"/refine"** after brainstorming and provides chosen approach

**What happens**:
1. AI asks 5-7 scope questions
2. User answers
3. AI asks follow-ups based on answers
4. User clarifies
5. AI generates refinement summary
6. User approves
7. Ready for `/breakdown`

**Next step**: Use refinement summary with `/breakdown` - breakdown Layer 3 will reference it for technical decisions
