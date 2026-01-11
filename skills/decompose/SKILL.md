---
name: decompose
description: Decomposes specifications and test requirements into executable tasks organized by feature areas. Creates unit-testable tasks for capable agents with specific test requirements, plus feature-level review tasks for integration verification.
model: claude-sonnet-4-20250514
---

# Decompose: From Spec to Executable Tasks

This skill transforms specifications and test requirements into concrete, executable tasks organized by feature areas. It bridges the gap between "what we're building" (spec) and "how we'll execute it" (tasks ready for implementation).

## Workflow Context

**This skill can be used:**
- **As part of the planning pipeline**: `/brainstorm` → `/breakdown` → `/spec` → `/test-reqs` → `/decompose`
- **Standalone**: When you have a spec + test requirements and need executable task breakdown

**When in the pipeline**: Takes output from `/spec` (formal specification) and `/test-reqs` (test scenarios) to generate feature areas with concrete tasks. **Important**: Reference `/breakdown` Layer 3 for technical decisions - use specific tech choices (PostgreSQL, JWT, Redis) in task descriptions rather than generic terms ("database", "auth token", "cache").

## Execution Rhythm: One Phase Per Turn

**IMPORTANT**: Do NOT rush through multiple phases in one response. Each phase should be its own turn:

1. **Phase 1 (Context Review)**: Review spec + test-reqs → STOP → Confirm understanding
2. **Phase 2 (Feature Analysis)**: Analyze features and complexity → STOP → Get feedback
3. **Phase 3 (Task Generation)**: Propose task breakdown → STOP → Allow refinement
4. **Phase 4 (Finalization)**: Present final JSON → STOP → Confirm ready for project creation

This prevents overwhelming the user and allows course-correction at each stage.

## Core Principles

**Principles for task decomposition**:

1. **One spec FEATURE = one feature area** - Maintains feature cohesion
2. **Tasks sized for capable agents** - Workers are like full chat sessions, use their potential
3. **Test requirements are natural language but SPECIFIC** - No room for testing rig to misinterpret
4. **Only review tasks are `[testing]` tasks** - Implementation tasks have test requirements but aren't testing tasks
5. **Review = feature-level integration checkpoint** - Verify tasks work together before milestone integration
6. **Parallel when possible** - Dependencies handled by project planning system

**Important distinction**:
- **Implementation tasks**: Have test requirements, worker self-tests, testing rig verifies, loops on failure
- **Review tasks**: Only actual `[testing]` tasks, feature-level integration verification

---

## The Process

### Phase 1: Context Review
**Goal**: Understand the spec and test requirements

**What to review**:
- **Spec sections**:
  - OVERVIEW: What are we building?
  - GOALS: Why are we building it?
  - FEATURES: List of features with capabilities, edge cases
  - DATA & STATE: Entities and state transitions
  - DEPENDENCIES: What this relies on
  - SUCCESS CRITERIA: How we know it works
  - EDGE CASES & CONSTRAINTS: System-level failure modes

- **Test-reqs sections**:
  - Phase 1 (Execution Mockup): Behavioral walkthrough
  - Phase 2 (Test Scenarios): BDD scenarios organized by methodology
  - Test priorities (P0/P1/P2)
  - Coverage summary

**Ask clarifying questions if**:
- Spec features are ambiguous
- Test scenarios don't map clearly to features
- Dependencies are unclear
- Edge cases seem incomplete

**Output**: Clear understanding of what needs to be built and tested

---

### Phase 2: Feature Analysis
**Goal**: Analyze each feature to determine task boundaries

For each spec FEATURE, perform 6-component analysis:

#### Component 1: Spec Analysis
**Extract from spec**:
- Feature capabilities (from "What" and "How it works")
- Edge cases per feature
- Data entities touched (from DATA & STATE)
- Dependencies (from DEPENDENCIES + infer from data flow)

**Example**:
```
Feature: Email/Password Login
  Capabilities:
    - Credential validation
    - Authentication check
    - Session creation
    - Failed attempt tracking
  Entities: User, Session
  Dependencies: Database, hashing library
```

#### Component 2: Test Coverage Analysis
**Extract from test-reqs**:
- Test scenarios for this feature
- Test methodologies (happy path, edge case, error path, UI flow)
- Priorities (P0/P1/P2)
- Specific assertions

**Example**:
```
Feature: Email/Password Login
  Scenarios:
    - [P0, Happy Path] Valid login → dashboard
    - [P0, Error Path] Invalid credentials → generic error
    - [P1, Edge Case] Empty fields → validation
    - [P1, Edge Case] SQL injection → sanitized
```

#### Component 3: Task Boundary Inference
**Use intelligent heuristics**:

**Heuristic 1 - Natural Capabilities**:
- Each distinct capability = candidate task
- Simple capability (1 entity, 1-2 edge cases) → one task
- Complex capability (multiple entities, 5+ edge cases) → split by sub-capability

**Heuristic 2 - Test Clustering**:
- Group test scenarios by what they test
- If 3+ scenarios test same thing → that thing is likely a task

**Heuristic 3 - Complexity Scoring**:
```
Complexity = (# edge cases × 2) + (# entities × 3) + (# external deps × 5)

Score 0-5: Simple → 1 task (15-30 min)
Score 6-15: Medium → 2-3 tasks (30 min each)
Score 16+: Complex → 4+ tasks (30-45 min each)
```

**Heuristic 4 - Agent Capability Respect**:
- **Don't over-split**: File-level work stays together ("Create login endpoint" not "Create skeleton" + "Add validation" + "Add error handling")
- **Do split**: When crossing architectural boundaries (API + UI = 2 tasks)
- **Do split**: When distinct testing needed (separate concerns)

**Example decision**:
```
Feature: Email/Password Login
  Complexity: 12 (4 edge cases × 2 + 1 entity × 3 + 0 deps × 5)
  → Medium → 2-3 implementation tasks + 1 review task

  Proposed:
    1. Login endpoint with validation/auth (30 min)
    2. Failed attempt tracking (30 min)
    3. Session management (30 min)
    4. Review task (45 min)
```

#### Component 4: Task Generation
**For each implementation task, generate**:

**Description** (imperative, specific):
- Format: "[Action] [What] [Context/Detail]"
- Example: "Create POST /api/auth/login endpoint with email/password validation and authentication check"

**Test Requirements** (natural language, specific, no ambiguity):
- What to verify (observable outcomes)
- Specific values/behaviors
- Format testing rig can execute
- Example:
  ```
  - POST /api/auth/login with valid credentials returns 200 status
  - Response includes session token in secure HTTP-only cookie
  - Invalid credentials return 401 with message "Invalid credentials"
  - Email validation rejects malformed emails before database check
  ```

**Time Estimate** (based on complexity):
- 15 min: Simple (add field, small config, obvious logic)
- 30 min: Standard (create endpoint, component, service logic) - most common
- 45 min: Complex (multi-step logic, integration, error handling)

#### Component 5: Review Task Generation
**For each feature, add ONE review task**:

**Description**: `[testing] Review [Feature Name] - feature integration verification`

**Test Requirements**: All P0/P1 test scenarios for this feature
- Run all P0 scenarios (must pass)
- Run all P1 scenarios (should pass)
- Verify end-to-end integration
- Verify tasks work together correctly

**Time Estimate**: 30-45 min (testing takes time)

**Verification Type**: "test" (vs "build" for implementation tasks)

**Example**:
```
Task: "[testing] Review Email/Password Login - feature integration verification"
Test Requirements:
  - Run all P0 scenarios: valid login succeeds, invalid fails with generic error
  - Run all P1 scenarios: empty fields validated, malformed email rejected
  - Verify end-to-end: user logs in, receives session, accesses protected pages
  - Verify failed attempt tracking integrates with login flow
Time: 45 min
```

#### Component 6: Tag Selection
**Auto-select tags**:

**System tags** (detect from language):
- "endpoint", "API", "route" → `api`
- "component", "page", "UI" → `ui`
- "schema", "migration", "table" → `database`
- "auth", "permission", "validation" → `security`
- etc.

**Feature tag** (required on every feature area):
- Format: `{feature-name}-feature`
- Example: `auth-feature`, `export-feature`
- Same tag across ALL related feature areas

**Special tag** (required on LAST feature area only):
- `feature-complete` - Triggers review when all tasks complete

**Output**: Comprehensive feature analysis with proposed tasks

---

### Phase 3: Task Generation & Presentation
**Goal**: Generate complete task breakdown and present for refinement

**Generate JSON structure**:
```json
{
  "feature_areas": [
    {
      "name": "[Feature Name from spec]",
      "tags": ["system-tag", "feature-name-feature"],
      "tasks": [
        {
          "description": "Imperative task description",
          "test_requirements": [
            "Specific testable requirement 1",
            "Specific testable requirement 2"
          ],
          "time_estimate": 30
        }
      ],
      "test_acceptance_criteria": [
        "Feature-level acceptance criterion 1",
        "Feature-level acceptance criterion 2"
      ],
      "dependencies": ["Other Feature Name"]
    }
  ],
  "parallel_tracks": [
    ["Feature 1", "Feature 2"],
    ["Feature 3"]
  ]
}
```

**Present to user**:
1. Show proposed feature areas
2. Show tasks per feature with rationale
3. Show dependencies
4. Show parallel tracks
5. **Ask for feedback**: "Should I split any tasks? Merge any? Adjust dependencies?"

**Allow refinement**:
- User can request splitting tasks ("Split task 2 into smaller pieces")
- User can request merging tasks ("Combine tasks 1 and 2")
- User can adjust dependencies
- User can change time estimates

**Output**: Refined task breakdown ready for finalization

---

### Phase 4: Finalization
**Goal**: Present final JSON ready for project creation

**Validation checklist**:
- [ ] All feature areas have at least one system tag
- [ ] All feature areas have `{name}-feature` tag
- [ ] Only LAST feature area has `feature-complete` tag
- [ ] Dependencies reference exact feature area names
- [ ] No circular dependencies
- [ ] All tasks have description and time_estimate (15, 30, or 45)
- [ ] Test requirements are specific and unambiguous
- [ ] Review tasks prefixed with `[testing]`
- [ ] Parallel tracks correctly reflect dependency structure

**Output**: Final JSON ready for project planning

---

## Output Format

### Complete Structure

```json
{
  "feature_areas": [
    {
      "name": "Feature Name",
      "tags": ["system-tag", "feature-name-feature"],
      "tasks": [
        {
          "description": "Create endpoint with validation",
          "test_requirements": [
            "Endpoint returns 200 on valid input",
            "Endpoint returns 400 on invalid input with specific error message"
          ],
          "time_estimate": 30
        },
        {
          "description": "[testing] Review Feature Name - feature integration verification",
          "test_requirements": [
            "Run all P0 test scenarios for this feature",
            "Run all P1 test scenarios for this feature",
            "Verify end-to-end feature works as specified"
          ],
          "time_estimate": 45
        }
      ],
      "test_acceptance_criteria": [
        "Feature-level criterion 1 from success criteria",
        "Feature-level criterion 2 from success criteria"
      ],
      "dependencies": []
    }
  ],
  "parallel_tracks": [
    ["Feature 1"],
    ["Feature 2"]
  ]
}
```

### Field Specifications

**Feature Area Fields**:
- `name`: Feature name from spec FEATURES section
- `tags`: Array of system tags + feature tag (+ feature-complete on last area)
- `tasks`: Array of task objects
- `test_acceptance_criteria`: Feature-level success criteria from spec
- `dependencies`: Array of feature area names that must complete first

**Task Fields**:
- `description`: Imperative mood, specific, includes context
- `test_requirements`: Array of natural language, specific test requirements
- `time_estimate`: 15, 30, or 45 (minutes)

**Parallel Tracks**:
- Array of arrays grouping features by dependency depth
- Track 1: No dependencies (start immediately)
- Track 2: Depends only on Track 1
- Track 3: Depends on Track 2, etc.

---

## System Tags Reference

**Use these exact tags - they have colors defined in the UI:**

| Tag | Color | Use For |
|-----|-------|---------|
| `setup` | Gray | Initial configuration, scaffolding |
| `architecture` | Blue | Core patterns, abstractions |
| `database` | Amber | Schema changes, migrations |
| `api` | Green | Backend endpoints, services |
| `ui` | Pink | Frontend components, pages |
| `testing` | Cyan | Test tasks (review tasks only) |
| `security` | Red | Auth, validation, permissions |
| `schema` | Violet | Database schema work |
| `mcp` | Orange | MCP server tools |

**Feature Tag** (teal color):
- Format: `{feature-name}-feature`
- Example: `auth-feature`, `export-feature`

**Special Tag** (purple color):
- `feature-complete` - On LAST feature area only

---

## Complete Example

### Input: Spec + Test-Reqs

**Spec Feature**:
```markdown
### Feature A: Task Export to CSV
**What**: Export tasks to CSV format for data portability

**Why**: Users need to backup data and use it in external tools

**How it works**:
- User clicks "Export" button on tasks page
- System generates CSV with all visible tasks
- Browser downloads file named "tasks-YYYY-MM-DD.csv"
- CSV includes columns: Title, Status, Created Date, Due Date

**Success looks like**:
- User can download CSV file
- CSV opens in Excel/Google Sheets
- All task data is present and correct

**Edge cases**:
- No tasks: Show "No tasks to export" message
- Special characters in title: Properly escaped in CSV
- Missing due date: Empty cell (not "null")
```

**Test-Reqs Scenarios**:
```gherkin
Scenario: Export all tasks to CSV [P0, Happy Path]
  Given user has 3 tasks
  When user clicks "Export"
  Then CSV downloads with 3 rows

Scenario: CSV properly escapes special characters [P1, Edge Case]
  Given task titled: Buy milk, eggs, and "cheese"
  When user exports to CSV
  Then title is properly escaped

Scenario: Export with no tasks [P1, Error Path]
  Given user has 0 tasks
  When user clicks "Export"
  Then message shows "No tasks to export"
```

### Output: Decomposed Tasks

```json
{
  "feature_areas": [
    {
      "name": "Task Export to CSV",
      "tags": ["api", "ui", "export-feature", "feature-complete"],
      "tasks": [
        {
          "description": "Create POST /api/tasks/export endpoint that generates CSV from user's tasks",
          "test_requirements": [
            "POST /api/tasks/export returns 200 with CSV content",
            "CSV includes headers: Title, Status, Created Date, Due Date",
            "CSV contains all tasks visible to authenticated user",
            "CSV properly escapes special characters (commas, quotes) in task titles",
            "Tasks with missing due dates show empty cell (not 'null' or 'undefined')",
            "Response has Content-Type: text/csv and Content-Disposition: attachment"
          ],
          "time_estimate": 30
        },
        {
          "description": "Add Export button to tasks page with download trigger",
          "test_requirements": [
            "Export button visible in tasks page header",
            "Clicking Export triggers API call to /api/tasks/export",
            "Browser downloads file named 'tasks-YYYY-MM-DD.csv'",
            "When user has 0 tasks, button shows message 'No tasks to export'",
            "Button shows loading state during export"
          ],
          "time_estimate": 30
        },
        {
          "description": "[testing] Review Task Export to CSV - feature integration verification",
          "test_requirements": [
            "Run P0 scenario: User with 3 tasks exports successfully and CSV contains all data",
            "Run P1 scenario: Special characters in titles are properly escaped in CSV",
            "Run P1 scenario: User with 0 tasks sees 'No tasks to export' message",
            "Verify end-to-end: User clicks Export, file downloads, opens in Excel correctly",
            "Verify CSV format follows RFC 4180 standard"
          ],
          "time_estimate": 45
        }
      ],
      "test_acceptance_criteria": [
        "User can download CSV file with task data",
        "CSV opens correctly in Excel and Google Sheets",
        "All task data present and properly formatted",
        "Special characters handled correctly",
        "Empty state handled gracefully"
      ],
      "dependencies": []
    }
  ],
  "parallel_tracks": [
    ["Task Export to CSV"]
  ]
}
```

---

## Integration with Workflow Pipeline

```
/brainstorm → /breakdown → /spec → /test-reqs → /decompose
                   ↓          ↓         ↓            ↓
              Layer 3     Formal    Test Reqs   Feature Areas
              Blueprint    Spec                  + Tasks
```

**How they connect**:
- `/breakdown` Layer 3: Technical blueprint with architecture
- `/spec`: Formalizes into structured specification (what + why)
- `/test-reqs`: Generates execution mockup + test scenarios
- `/decompose`: Combines spec + test-reqs → executable tasks

**Next step after decompose**:
- Use output with project planning tools
- Create milestone with feature areas and tasks
- Workers execute tasks with test requirements
- Review tasks verify feature integration

---

## Tips for Quality Decomposition

### Feature Analysis
- Map spec features 1:1 to feature areas (maintain cohesion)
- Don't reorganize by tech stack unless feature naturally splits that way
- Respect natural boundaries from spec

### Task Sizing
- **Good size**: Worker can complete + self-test in 15-45 min
- **Too small**: "Add import statement" - micromanaging the worker
- **Too large**: "Build entire auth system" - needs breakdown
- **Just right**: "Create login endpoint with validation and error handling"

### Test Requirements
- **Be specific**: "Returns 200 with session token" not "Works correctly"
- **Use concrete values**: "Message: 'Invalid credentials'" not "Shows error"
- **No ambiguity**: Testing rig should know exactly what to check
- **Observable outcomes**: What can be verified, not internal implementation

### Review Tasks
- **One per feature**: Don't split review across multiple tasks
- **All P0/P1 scenarios**: Comprehensive feature verification
- **Integration focus**: Verify tasks work together, not individual tasks
- **Last task in feature**: Placed after all implementation tasks

### Dependencies
- **Explicit when needed**: Data layer before API layer
- **Parallel when possible**: Multiple UI features can be parallel
- **No artificial deps**: Don't add dependency unless truly needed
- **Use spec DEPENDENCIES**: Trust what spec says about external deps

---

## Common Pitfalls to Avoid

1. **Over-splitting tasks**: "Create file" + "Add function" + "Add tests" → Just "Create service with tests"
2. **Under-splitting tasks**: "Build entire feature" → Break into 2-4 logical units
3. **Vague test requirements**: "Test that it works" → "POST returns 200 with valid JSON response"
4. **Creating testing tasks for implementation**: Only review tasks are `[testing]`
5. **Missing review tasks**: Every feature needs integration verification
6. **Wrong tag on feature-complete**: Must be on LAST feature only
7. **Circular dependencies**: A depends on B, B depends on A → Restructure
8. **Dependency name typos**: Must match feature area name exactly

---

## Usage

**Starting the skill**:
User says: **"/decompose"** and provides:
- Spec (from `/spec` output or user-written)
- Test-reqs (from `/test-reqs` output)

**Process**:
1. Review context (spec + test-reqs)
2. Analyze features and propose task breakdown
3. Get user feedback and refine
4. Finalize JSON output

**What you get**:
- Complete feature area breakdown
- Tasks with specific test requirements
- Review tasks for integration verification
- Dependencies and parallel tracks
- JSON ready for project creation

**Next steps**:
- Use output with project planning
- Create milestone + tasks
- Workers execute with test requirements
- Review tasks verify integration
