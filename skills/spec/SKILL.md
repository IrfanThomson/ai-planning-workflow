---
name: spec
description: Creates detailed, behavior-focused specifications from breakdown output. Describes WHAT needs to exist and WHY, not HOW to implement it. Generates structured spec with features, edge cases, success criteria, and dependencies.
model: claude-sonnet-4-20250514
---

# Spec: High-Level Specification Creation

This skill creates detailed, behavior-focused specifications from breakdown output. The spec describes WHAT needs to exist and WHY, not HOW to implement it.

## Workflow Context

**This skill can be used:**
- **As part of the planning pipeline**: `/brainstorm` → `/breakdown` → `/spec` → `/test-reqs` → `/decompose`
- **Standalone**: When you need a formal specification for any feature or system

**When in the pipeline**: Takes `/breakdown` Layer 3 (blueprint) and formalizes it into a structured specification that feeds into `/test-reqs`. **Important**: Reference Layer 3 for technical decisions (tech stack, architecture patterns, data schemas) - these should be preserved in the spec's technical context, not abstracted away.

## Execution Rhythm: Generate Spec in One Turn

**This skill generates the complete spec in a single turn** after reviewing the breakdown output. The structured template format provides organization, so there's no need to pause between sections.

**User can**:
- Provide breakdown output → receive complete spec
- Request refinements → spec gets updated
- Ask clarifying questions during generation if needed

## Core Principle

**Behavior Over Implementation**: Describe what the system does, not how it's built. The spec should generate good test requirements and enable task decomposition without being prescriptive about implementation details.

## When to Use This Skill

Use this skill after completing a breakdown (from brainstorm or direct feature description). You have:
- Feature areas identified
- Rough understanding of what needs to be built
- Goals and context

You need:
- Detailed specification of behavior
- Clear success criteria
- Edge cases identified
- Foundation for test requirements generation

## The Process

### Phase 1: Context Review
**Goal**: Understand what was decided in breakdown

Review:
- **Breakdown output**: Feature areas, rough descriptions, dependencies
- **Original goals**: What problem are we solving?
- **Constraints**: Timeline, technical limitations, dependencies
- **Target user/actor**: Who uses this? What's their context?

**Output**: Clear understanding of scope and intent

---

### Phase 2: Spec Generation
**Goal**: Create structured specification following the template

Generate spec with these sections:

#### 1. OVERVIEW
- 1-2 paragraphs describing what this is
- Optional analogy if helpful
- Should be understandable by anyone (non-technical too)

#### 2. GOALS
- Why are we building this?
- What problem does it solve?
- Focus on impact, not tasks
- 3-5 bullet points maximum

#### 3. SUCCESS CRITERIA
- How do we know it works?
- Observable outcomes (not process checkpoints)
- Specific and testable
- Example: "User can export data to CSV and open it in Excel" not "Export feature is implemented"

#### 4. HOW IT WORKS
- Step-by-step narrative flow
- From user/actor perspective
- Happy path walkthrough
- NOT implementation details (no "create API endpoint", just "system validates credentials")

#### 5. FEATURES
For each feature area from breakdown:

**Feature Name: [Capability description]**

**What**: What capability does this provide?

**Why**: Why does this matter? What's the impact/value?

**How it works**: Behavioral flow description
- What happens when user/system does X?
- State transitions
- NOT implementation ("API returns JSON" → "System responds with user data")

**Success looks like**: Observable outcomes specific to this feature
- What can you see/verify?
- Specific assertions

**Edge cases**: Error states, boundary conditions
- What happens when things go wrong?
- What are the limits?
- Failure modes

#### 6. DATA & STATE
- What gets stored?
- What changes over time?
- Entity descriptions (NOT database schemas)
- State transitions
- Relationships between entities

Example: "System tracks user sessions. Each session has: user, expiration time, permissions. When session expires, user must re-authenticate."

NOT: "sessions table with columns: id, user_id, expires_at, permissions JSONB"

#### 7. EDGE CASES & CONSTRAINTS
- System-level failure modes not covered in individual features
- Cross-cutting concerns
- Performance constraints
- Security constraints
- What happens when external dependencies fail?

#### 8. OUT OF SCOPE
- Explicitly state what we're NOT building
- Prevents scope creep
- Sets boundaries
- Clarifies misunderstandings early

Example: "Out of scope: Password reset via SMS (email only for v1)"

#### 9. DEPENDENCIES
- What does this rely on? (External services, other features, infrastructure)
- What relies on this? (Downstream impact)
- Assumptions about what exists

**Output**: Complete specification document

---

### Phase 3: Validation & Refinement
**Goal**: Ensure spec is complete and unambiguous

Check:
- **Testability**: Can you generate test scenarios from this?
  - Are success criteria specific enough?
  - Are edge cases enumerated?
  - Can you tell when it's "done"?

- **Decomposability**: Can you break this into tasks?
  - Are features clear enough to implement?
  - Is behavior well-defined?
  - Are dependencies explicit?

- **Completeness**: Is anything missing?
  - What happens when things go wrong?
  - What about edge cases?
  - Are there unstated assumptions?

- **Clarity**: Can someone else understand this?
  - Is jargon explained?
  - Are flows clear?
  - Is the "why" present?

**Ask clarifying questions** if:
- Success criteria are vague
- Edge cases are unclear
- Features have ambiguous behavior
- Dependencies are unstated

**Output**: Validated, complete spec ready for test-reqs and task decomposition

---

## Spec Template

```markdown
# [Feature/Project Name]

## 1. OVERVIEW
[1-2 paragraphs describing what this is. Optional analogy.]

## 2. GOALS
Why are we building this? What problem does it solve?
- Goal 1: [Impact-focused]
- Goal 2: ...

## 3. SUCCESS CRITERIA
How do we know it works? (Observable outcomes)
- Criterion 1: [Specific, testable]
- Criterion 2: ...

## 4. HOW IT WORKS
[Step-by-step narrative flow from user/actor perspective. Happy path.]

## 5. FEATURES

### Feature A: [Name]
**What**: [Capability description]

**Why**: [Impact, value]

**How it works**: [Behavioral flow]

**Success looks like**: [Observable outcomes]

**Edge cases**: [Error states, boundaries]

### Feature B: [Name]
...

## 6. DATA & STATE
What gets stored? What changes?
[Entities, relationships, state transitions - NOT schemas]

## 7. EDGE CASES & CONSTRAINTS
What happens when things go wrong?
[System-level failure modes, performance/security constraints]

## 8. OUT OF SCOPE
What we're NOT building (explicitly)
[Prevents scope creep, sets boundaries]

## 9. DEPENDENCIES
What does this rely on? What relies on this?
[External services, other features, assumptions]
```

---

## Adaptive Spec Types (Future Enhancement)

The template above is the default. Future versions may support type-aware specs:

- **UI-heavy features**: Emphasize user flows, interactions, visual states
- **API features**: Emphasize contracts, error codes, request/response patterns
- **Data pipelines**: Emphasize input/output formats, transformations, error handling
- **Infrastructure**: Emphasize components, dependencies, configuration

For now, use the default template and adapt sections as needed.

---

## Integration with Workflow Pipeline

```
/brainstorm → /refine → /breakdown → /spec → /test-reqs → /decompose
                   ↓         ↓
              Layer 3    Formal Spec
              Blueprint  (Behavior-focused)
```

**How they connect**:
- `/breakdown` Layer 3 provides technical blueprint
- `/spec` formalizes it into structured specification (what + why, not how)
- `/test-reqs` uses spec to generate execution mockup + test scenarios
- `/decompose` uses spec + test-reqs to create executable tasks

**The spec does NOT include**:
- Mockups (generated by test-reqs)
- Test scenarios (generated by test-reqs)
- Implementation details (database schemas, API endpoints, file paths)

---

## Tips for Good Specs

**As the spec writer:**
- **Stay high-level**: Describe behavior, not implementation
- **Be specific**: "User sees error message 'Invalid email'" not "System handles errors"
- **Think like a user**: What do they experience? What can they observe?
- **Enumerate edge cases**: Don't assume happy path only
- **Be explicit about scope**: "Out of scope" section is critical
- **Ask questions**: Better to clarify now than guess wrong

**Common mistakes:**
- ❌ Jumping to implementation ("Create a PostgreSQL table...")
- ❌ Vague success criteria ("It should work well")
- ❌ Ignoring edge cases (only happy path)
- ❌ Missing the "why" (no goals/context)
- ❌ Assuming knowledge (unstated dependencies)

---

## Example: User Authentication Feature

```markdown
# User Authentication System

## 1. OVERVIEW
A secure authentication system that allows users to log in with email/password, manages sessions, and enforces security policies. Think of it like a bouncer at a club - checks credentials, issues wristbands (sessions), and enforces access rules.

## 2. GOALS
- Allow legitimate users to access their accounts securely
- Prevent unauthorized access through robust security measures
- Provide clear feedback when authentication fails
- Maintain user sessions across page reloads

## 3. SUCCESS CRITERIA
- User can log in with valid credentials and access protected pages
- Invalid credentials show clear error message without revealing which part is wrong
- After 3 failed attempts, account locks for 15 minutes
- User sessions persist across browser refresh but expire after 24 hours
- User can log out and session is immediately invalidated

## 4. HOW IT WORKS
User visits login page and enters email and password. System validates credentials against stored data. If valid, system creates a session, stores it securely, and redirects user to dashboard. If invalid, system shows error message without revealing whether email or password was wrong (security). If user fails 3 times, system temporarily locks the account and shows lockout message with time remaining.

Once logged in, user can navigate the application. System checks session on each request. If session is valid and not expired, user proceeds. If expired, system redirects to login with message "Session expired, please log in again."

User can explicitly log out, which immediately destroys the session and redirects to login page.

## 5. FEATURES

### Feature A: Email/Password Login
**What**: Allows users to authenticate using email and password credentials.

**Why**: Primary authentication method for users to access their accounts.

**How it works**:
- User enters email and password on login form
- System validates format (email looks like email, password not empty)
- System checks credentials against stored data
- On match: Create session, set secure cookie, redirect to dashboard
- On mismatch: Show generic error "Invalid credentials" (don't reveal which field)
- Track failed attempts per email address

**Success looks like**:
- User with valid credentials reaches dashboard within 2 seconds
- Invalid credentials show error within 2 seconds
- Error message doesn't reveal whether email exists or password is wrong

**Edge cases**:
- Email doesn't exist → same error as wrong password
- Empty fields → client-side validation prevents submission
- Malformed email → client-side validation catches it
- SQL injection attempts → parameters are sanitized
- Password with special characters → handled correctly

### Feature B: Account Lockout
**What**: Temporarily locks accounts after repeated failed login attempts.

**Why**: Prevents brute force attacks while allowing legitimate users to recover.

**How it works**:
- System counts failed login attempts per email (resets on success)
- After 3 failures: Lock account for 15 minutes
- During lockout: All login attempts fail immediately with message "Too many failed attempts. Try again in X minutes."
- After 15 minutes: Lock automatically expires, counter resets
- Successful login before lockout: Counter resets

**Success looks like**:
- 3 failed attempts trigger lockout
- Lockout message shows accurate time remaining
- After 15 minutes, user can attempt login again
- Successful login resets failure counter

**Edge cases**:
- User tries correct password during lockout → still locked, show time remaining
- Multiple IPs trying same email → lockout applies to email, not IP
- Lockout expires mid-session → user can log in immediately

### Feature C: Session Management
**What**: Maintains authenticated state across requests and browser sessions.

**Why**: Users shouldn't have to log in on every page load. Sessions should be secure and expire appropriately.

**How it works**:
- On successful login: Generate unique session ID, store in secure HTTP-only cookie
- Session data stored server-side: user ID, created timestamp, expires timestamp (24h)
- On each request: Validate session ID, check expiration
- If valid: Continue request
- If expired: Clear cookie, redirect to login with message
- If invalid/missing: Treat as unauthenticated

**Success looks like**:
- User stays logged in across page refreshes
- Session expires exactly 24 hours after creation
- Expired session shows clear message and redirects to login
- Session cookie is HTTP-only and secure (not accessible via JavaScript)

**Edge cases**:
- Session cookie deleted by user → treated as logged out
- Session expired during active use → redirect to login with context message
- User logs in on multiple devices → each gets separate session
- Server restart → sessions persist (stored in database, not memory)

### Feature D: Logout
**What**: Allows user to explicitly end their session.

**Why**: Users need ability to log out, especially on shared devices.

**How it works**:
- User clicks logout button
- System invalidates session (delete from database)
- System clears session cookie
- System redirects to login page with message "You have been logged out"

**Success looks like**:
- After logout, user cannot access protected pages
- After logout, back button doesn't show cached protected pages (or shows with expired state)
- Logout completes within 1 second

**Edge cases**:
- Double logout (clicking logout twice) → no error, just redirects
- Logout with expired session → clears cookie anyway, shows success message
- Logout on one device → doesn't affect sessions on other devices

## 6. DATA & STATE
**User entity**: email (unique), password_hash, created_at, failed_login_attempts, locked_until (nullable timestamp)

**Session entity**: session_id (unique), user reference, created_at, expires_at

**State transitions**:
- User: unlocked ↔ locked (after 3 failures / after 15 min)
- Session: active → expired (after 24 hours or logout)

**Relationships**:
- One user can have multiple active sessions (multiple devices)
- Session belongs to exactly one user

## 7. EDGE CASES & CONSTRAINTS
- **Network failure during login**: User sees error, no session created
- **Database unavailable**: Show maintenance message, don't crash
- **Session cookie tampering**: Validate session server-side, reject if invalid
- **Password stored as hash**: Never store plaintext passwords
- **Rate limiting**: Consider adding request rate limits per IP (future enhancement)
- **Performance**: Login should complete within 2 seconds under normal load

## 8. OUT OF SCOPE
- Password reset (email-based recovery) - future feature
- OAuth/social login (Google, GitHub) - future feature
- Two-factor authentication - future feature
- Remember me (extended sessions) - future feature
- Account creation/registration - separate feature
- Password strength enforcement - future feature (current: no minimum requirements)

## 9. DEPENDENCIES
**Relies on**:
- Database for user and session storage
- Secure hashing library (bcrypt or similar) for password verification
- HTTP cookie mechanism for session transport
- Time/clock for session expiration (system time must be accurate)

**Relied upon by**:
- All protected pages/features (need authentication)
- User profile management
- Authorization system (roles/permissions)
```

---

## Handoff

When spec is complete and validated:
- User can review and approve
- Pass to `/test-reqs` for mockup and scenario generation
- Spec + test-reqs output feeds into `/decompose` for task creation

The spec becomes the source of truth for "what needs to exist and why."
