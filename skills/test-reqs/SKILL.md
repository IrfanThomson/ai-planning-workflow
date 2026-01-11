---
name: test-reqs
description: Generate comprehensive test requirements from specifications. Phase 1 creates execution mockup (behavioral walkthrough), Phase 2 generates test scenarios with intelligent methodology selection (edge cases, stress testing, UI flows, etc.).
model: claude-sonnet-4-20250514
---

# Test Requirements Generator

This skill generates comprehensive test requirements from specifications through a two-phase process: execution mockup generation followed by intelligent test scenario creation.

## Workflow Context

**This skill can be used:**
- **As part of the planning pipeline**: `/brainstorm` → `/breakdown` → `/spec` → `/test-reqs` → `/decompose`
- **Standalone**: When you have a spec and need test requirements for any feature

**When in the pipeline**: Takes spec output and generates test requirements (execution mockup + test scenarios) that feed into `/decompose`. **Tip**: If breakdown Layer 3 is available, reference it for implementation-specific test details (e.g., "Test JWT token format" vs generic "Test session token").

## Execution Rhythm: Two Phases, One Turn

**Unlike brainstorm**, this skill typically generates both phases in a single turn:
1. **Phase 1**: Execution Mockup (behavioral walkthrough)
2. **Phase 2**: Test Scenarios (methodology selection + BDD scenarios)

Both outputs are presented together as a complete test requirements document.

## Core Principle

**Spec → Behavior → Tests**: First understand what the system should DO (execution mockup), then determine how to TEST it (test scenarios with methodology selection).

---

## The Process

### Phase 1: Execution Mockup Generation
**Goal**: Create a complete behavioral walkthrough of the feature from the specification

**What to generate**:
- **Happy Path**: Primary user journey from start to finish
- **Alternative Paths**: Other valid ways to accomplish the goal
- **Error Paths**: What happens when things go wrong (validation errors, network failures, permissions)
- **Edge Cases**: Boundary conditions, empty states, maximum limits
- **User Experience**: Visual feedback, loading states, transitions
- **Data Flows**: What data moves where, transformations, persistence
- **State Changes**: How the system state evolves through the flow

**Format**: Structured sections with concrete examples:
```markdown
## Happy Path
[Step-by-step walkthrough of primary flow with specific examples]

## Alternative Paths
[Other valid user journeys]

## Error Scenarios
[What can go wrong and how system responds]

## Edge Cases
[Boundary conditions and unusual inputs]

## Data Flow
[Data transformations and persistence]

## State Transitions
[How system state changes]
```

**Key Principles**:
- **Be concrete**: Use specific examples, not abstract descriptions
- **Be thorough**: Don't skip error states or edge cases
- **Show state changes**: Make it clear what changes in the system
- **Include UX details**: Loading states, error messages, visual feedback
- **Think like a user**: Walk through the actual experience

---

### Phase 2: Test Scenario Generation
**Goal**: Analyze the execution mockup and generate comprehensive test scenarios with intelligent methodology selection

#### Step 1: Analyze Feature Characteristics

Examine the mockup to determine:
- **Feature Type**: CRUD operation, real-time interaction, batch process, multi-step workflow, etc.
- **Risk Factors**: Security concerns, data integrity, performance requirements, user impact
- **Complexity Level**: Simple form vs complex wizard vs distributed system
- **Integration Points**: External APIs, databases, third-party services
- **User Interaction**: UI-heavy vs API-only vs background job

#### Step 2: Select Testing Methodologies

Based on analysis, choose appropriate methodologies from:

| Methodology | When to Use | What to Test |
|-------------|-------------|--------------|
| **Happy Path Testing** | Always | Core functionality works end-to-end |
| **Edge Case Testing** | Boundary values, limits, empty states | Min/max values, null inputs, empty collections, extreme lengths |
| **Error Path Testing** | Validation, error handling | Invalid inputs, network failures, permission errors, conflict states |
| **UI Flow Testing** | User-facing features | Navigation, visual feedback, interactions, responsive behavior |
| **Stress Testing** | Performance-critical features | High load, concurrent users, large datasets, memory usage |
| **Security Testing** | Auth, data access, sensitive operations | Auth bypass, injection attacks, data leakage, privilege escalation |
| **Integration Testing** | External dependencies | API contracts, database transactions, third-party services |
| **State Testing** | Stateful systems | State transitions, consistency, race conditions |
| **Regression Testing** | Critical existing functionality | Ensure changes don't break existing behavior |

**Decision Framework**:

1. **Always include**: Happy Path, Error Path
2. **Feature Type triggers**:
   - CRUD → Edge Case Testing (boundary values)
   - Real-time → Stress Testing (concurrent users)
   - Multi-step → State Testing (transitions)
   - User-facing → UI Flow Testing (Playwright)
3. **Risk triggers**:
   - Auth/permissions → Security Testing
   - External APIs → Integration Testing
   - Performance SLA → Stress Testing
   - Data mutations → Regression Testing
4. **Complexity triggers**:
   - Simple form → Happy + Error + Edge
   - Complex wizard → + State + UI Flow
   - Distributed system → + Integration + Stress

#### Step 3: Generate Test Scenarios

For each selected methodology, generate specific scenarios in **BDD Given-When-Then format**:

```gherkin
Scenario: [Descriptive name]
  Given [Initial context/preconditions]
  When [Action taken]
  Then [Expected outcome]

  # Methodology: [Edge Case Testing]
  # Priority: [P0/P1/P2]
  # Automation: [Playwright/API/Unit]
```

**Priority Guidelines**:
- **P0**: Core functionality, data integrity, security (must pass for release)
- **P1**: Important flows, common edge cases (should pass for release)
- **P2**: Nice-to-have coverage, rare edge cases (optional)

#### Step 4: Add Playwright Hints (for UI scenarios)

For UI-heavy scenarios, include automation hints:
```
# Playwright Hints:
# - Click button: data-testid="submit-button"
# - Wait for: data-testid="success-message"
# - Assert visible: "Task created successfully"
# - Check redirect: URL should be /tasks/{id}
```

#### Step 5: Coverage Summary

Summarize test coverage:
```markdown
| Methodology | Scenarios | Priority Distribution |
|-------------|-----------|----------------------|
| Happy Path | 3 | P0: 3 |
| Edge Case | 5 | P1: 4, P2: 1 |
| Error Path | 6 | P0: 2, P1: 4 |
| UI Flow | 4 | P0: 1, P1: 3 |

**Total**: 18 scenarios
**P0**: 6 scenarios (must pass)
**P1**: 11 scenarios (should pass)
**P2**: 1 scenario (nice to have)
```

---

## Output Format

### Complete Output Template

```markdown
# Test Requirements: [Feature Name]

---

## Phase 1: Execution Mockup

### Happy Path
[Detailed walkthrough with specific examples]

### Alternative Paths
[Other valid flows]

### Error Scenarios
[What can go wrong]

### Edge Cases
[Boundary conditions]

### Data Flow
[Data transformations]

### State Transitions
[State changes]

---

## Phase 2: Test Scenarios

### Methodology Selection

**Feature Analysis:**
- Type: [CRUD/Real-time/Batch/etc]
- Risk: [Security/Performance/Data Integrity]
- Complexity: [Simple/Medium/Complex]
- UI: [Yes/No]

**Selected Methodologies:**
✓ Happy Path Testing - Core functionality
✓ Edge Case Testing - Boundary values
✓ Error Path Testing - Validation and errors
✓ UI Flow Testing - User interactions
✗ Stress Testing - Not performance-critical

**Justification**: [Brief explanation of why these methodologies were chosen]

---

### Test Scenarios

#### Happy Path Testing

**Scenario: [Descriptive name]**
```gherkin
Given [context]
When [action]
Then [outcome]

# Methodology: Happy Path Testing
# Priority: P0
# Automation: Playwright

# Playwright Hints:
# - [Specific automation guidance]
```

[... more scenarios for each methodology ...]

---

### Coverage Summary

| Methodology | Scenarios | Priority Distribution |
|-------------|-----------|----------------------|
| Happy Path | 3 | P0: 3 |
| Edge Case | 5 | P1: 4, P2: 1 |

**Total**: X scenarios
**P0**: X scenarios (must pass)
**P1**: X scenarios (should pass)
**P2**: X scenario (nice to have)
```

---

## Execution Guidelines

### Writing Execution Mockups
- **Think through the feature as if you're the user**
- **Don't forget error states** (they're often overlooked)
- **Include loading states and transitions**
- **Be specific about error messages and feedback**
- **Use concrete examples** (not "a user creates a task" but "user creates task titled 'Buy groceries'")

### Selecting Methodologies
- **Don't over-test simple features** (not everything needs stress testing)
- **Don't under-test risky features** (security, data integrity need thorough coverage)
- **Balance coverage with pragmatism** (P2 scenarios are optional)
- **Consider automation cost** (complex Playwright tests vs simple API tests)
- **Always justify selections** (explain why certain methodologies are included/excluded)

### Writing Test Scenarios
- **One assertion per scenario** (atomic tests)
- **Use concrete examples** (not abstract descriptions)
- **Make scenarios independent** (can run in any order)
- **Tag everything** (methodology, priority, automation type)
- **Provide enough context** (Given section should set up clearly)
- **Be specific in Then statements** (exact expected outcomes)

### Playwright Hints
- **Include specific selectors** (data-testid attributes)
- **Note timing considerations** (waits, animations)
- **Specify visual assertions** (what should be visible)
- **Include navigation expectations** (URL changes, redirects)

---

## Example: Full Workflow

### Input Spec
"Add ability to export tasks to CSV format"

### Output

```markdown
# Test Requirements: Task CSV Export

---

## Phase 1: Execution Mockup

### Happy Path
1. User navigates to tasks page showing 3 tasks: "Buy milk", "Walk dog", "Code review"
2. User clicks "Export" button in the page header
3. System generates CSV file containing all 3 visible tasks
4. Browser automatically downloads file named "tasks-2026-01-10.csv"
5. CSV file contains headers: Title, Status, Created Date, Due Date
6. Each task appears as a row with properly formatted data

### Alternative Paths
**Export Filtered Tasks**:
- User applies filter "Status: Active" (showing 2 of 3 tasks)
- User clicks "Export"
- CSV contains only the 2 filtered tasks

**Export from Different Views**:
- User can export from "Today" view, "Upcoming" view, or "All" view
- Export respects current view filter

### Error Scenarios
**No Tasks Available**:
- User has zero tasks in current view
- User clicks "Export"
- System shows message: "No tasks to export"
- No file downloads

**Export Generation Fails**:
- Network error or server issue during export
- System shows error: "Failed to generate export. Please try again."
- User can retry the operation

### Edge Cases
**Special Characters in Title**:
- Task titled: `Buy milk, eggs, and "cheese"`
- CSV properly escapes: `"Buy milk, eggs, and ""cheese"""`

**Missing Due Date**:
- Task has no due date set
- CSV shows empty cell (not "null" or "undefined")

**Large Dataset**:
- User has 1000+ tasks
- Export completes within reasonable time (< 10 seconds)
- All tasks included in CSV

**Very Long Title**:
- Task title is 255 characters (maximum allowed)
- Full title preserved in CSV without truncation

### Data Flow
1. Frontend: User clicks Export → sends GET `/api/tasks/export?view=active`
2. Backend: Receives request with user auth token
3. Backend: Queries database for tasks matching view filter + user permissions
4. Backend: Generates CSV in memory with proper escaping
5. Backend: Returns file with headers: `Content-Type: text/csv`, `Content-Disposition: attachment; filename="tasks-{timestamp}.csv"`
6. Frontend: Browser triggers download dialog

### State Transitions
- **No state change** (read-only operation)
- No tasks are modified or created
- User can export multiple times without side effects

---

## Phase 2: Test Scenarios

### Methodology Selection

**Feature Analysis:**
- Type: Data export (read-only CRUD operation)
- Risk: Data integrity (CSV format correctness), file generation reliability
- Complexity: Medium (CSV generation with special character escaping, filtering logic)
- UI: Yes (button click, download trigger)
- Integration: Database query, file generation

**Selected Methodologies:**
✓ Happy Path Testing - Basic export functionality works
✓ Edge Case Testing - Special characters, empty states, large datasets, boundary values
✓ Error Path Testing - No tasks, server failures, permission errors
✓ UI Flow Testing - Button interaction, download trigger, visual feedback
✓ Integration Testing - API contract, CSV format validation, database queries
✗ Stress Testing - Not performance-critical for typical use (covered by large dataset edge case)
✗ Security Testing - Read-only operation, permissions handled by existing auth middleware
✗ State Testing - No state changes (read-only)

**Justification**: Focus on data integrity (CSV format) and user experience (UI flow). Edge cases critical for CSV generation (escaping, empty values). Skip stress testing as load is not expected; skip security as auth is handled upstream.

---

### Test Scenarios

#### Happy Path Testing

**Scenario: Export all tasks to CSV**
```gherkin
Given user has 3 tasks in their account
  And tasks are: "Buy milk", "Walk dog", "Code review"
  And user is on the tasks page
When user clicks "Export" button
Then CSV file should download automatically
  And file name should match pattern "tasks-YYYY-MM-DD.csv"
  And CSV should contain exactly 3 task rows (plus header)
  And CSV headers should be: "Title,Status,Created Date,Due Date"
  And all 3 tasks should appear in CSV with correct data

# Methodology: Happy Path Testing
# Priority: P0
# Automation: Playwright + File validation

# Playwright Hints:
# - Click: data-testid="export-button"
# - Wait for download: const download = await page.waitForEvent('download')
# - Get file path: const path = await download.path()
# - Validate CSV: Read file, parse, assert row count = 3, assert headers
```

**Scenario: Export respects current view filter**
```gherkin
Given user has 5 tasks total
  And 3 tasks are "Active" status
  And 2 tasks are "Completed" status
  And user filters view to "Status: Active"
When user clicks "Export" button
Then CSV should contain exactly 3 task rows
  And all exported tasks should have "Active" status
  And "Completed" tasks should not appear in export

# Methodology: Happy Path Testing
# Priority: P0
# Automation: API + CSV validation
```

---

#### Edge Case Testing

**Scenario: Task title contains comma and quotes**
```gherkin
Given user has a task titled: Buy milk, eggs, and "cheese"
When user exports tasks to CSV
Then CSV should properly escape the title
  And parsed title should exactly match: Buy milk, eggs, and "cheese"
  And CSV cell should be: "Buy milk, eggs, and ""cheese"""

# Methodology: Edge Case Testing
# Priority: P1
# Automation: API + CSV parsing validation
```

**Scenario: Task has no due date**
```gherkin
Given user has a task with no due date set
When user exports tasks to CSV
Then due date cell should be empty string
  And CSV should not contain "null" or "undefined"
  And CSV row should still have correct number of columns

# Methodology: Edge Case Testing
# Priority: P1
# Automation: API + CSV parsing
```

**Scenario: Export with 1000+ tasks**
```gherkin
Given user has 1500 tasks in their account
When user clicks "Export" button
Then export should complete within 10 seconds
  And CSV should contain all 1500 task rows
  And file should download successfully
  And no timeout errors should occur

# Methodology: Edge Case Testing
# Priority: P2
# Automation: API + Performance check
```

**Scenario: Task title at maximum length (255 characters)**
```gherkin
Given user has a task with 255-character title
When user exports tasks to CSV
Then full title should be preserved without truncation
  And CSV parsing should succeed
  And title should be exactly 255 characters

# Methodology: Edge Case Testing
# Priority: P1
# Automation: API + CSV validation
```

---

#### Error Path Testing

**Scenario: Export when no tasks exist**
```gherkin
Given user has 0 tasks in their account
  And user is on the tasks page
When user clicks "Export" button
Then user should see message "No tasks to export"
  And no file should download
  And export button should remain enabled

# Methodology: Error Path Testing
# Priority: P1
# Automation: Playwright
```

**Scenario: Export fails due to server error**
```gherkin
Given export endpoint returns 500 error
  And user is on tasks page
When user clicks "Export" button
Then user should see error message "Failed to generate export. Please try again."
  And no file should download
  And user should be able to retry export

# Methodology: Error Path Testing
# Priority: P1
# Automation: Playwright + API mocking
```

---

#### UI Flow Testing

**Scenario: Export button shows loading state**
```gherkin
Given user is on tasks page with tasks available
When user clicks "Export" button
Then button should show loading indicator immediately
  And button should be disabled during export
  And button text should change to "Exporting..."
When export completes successfully
Then button should return to normal state
  And button should show "Export" text again
  And button should be enabled

# Methodology: UI Flow Testing
# Priority: P1
# Automation: Playwright
```

**Scenario: Download triggers browser save dialog**
```gherkin
Given user is on tasks page
When user clicks "Export" and file generates
Then browser should trigger download
  And default filename should be "tasks-{today's date}.csv"
  And user should be able to save file

# Methodology: UI Flow Testing
# Priority: P0
# Automation: Playwright (verify download event)
```

---

#### Integration Testing

**Scenario: CSV format follows RFC 4180 standard**
```gherkin
Given user has tasks with various data (commas, quotes, newlines)
When user exports tasks
Then CSV should use UTF-8 encoding
  And CSV should use comma as delimiter
  And CSV should use double-quote as escape character
  And CSV should include header row
  And all rows should have consistent column count

# Methodology: Integration Testing
# Priority: P0
# Automation: API + CSV format validation
```

**Scenario: Date fields use ISO 8601 format**
```gherkin
Given user has tasks with created dates and due dates
When user exports tasks
Then created date should be formatted as "YYYY-MM-DD"
  And due date should be formatted as "YYYY-MM-DD"
  And dates should be parseable by standard libraries

# Methodology: Integration Testing
# Priority: P0
# Automation: API + Date parsing validation
```

---

### Coverage Summary

| Methodology | Scenarios | Priority Distribution |
|-------------|-----------|----------------------|
| Happy Path | 2 | P0: 2 |
| Edge Case | 4 | P1: 3, P2: 1 |
| Error Path | 2 | P1: 2 |
| UI Flow | 2 | P0: 1, P1: 1 |
| Integration | 2 | P0: 2 |

**Total**: 12 scenarios
**P0**: 5 scenarios (must pass for release)
**P1**: 6 scenarios (should pass for release)
**P2**: 1 scenario (nice to have)

**Coverage Assessment**:
- ✅ Core export functionality thoroughly tested
- ✅ CSV edge cases (escaping, empty values) covered
- ✅ User experience (loading states, error handling) validated
- ✅ Data integrity (format compliance) ensured
- ⚠️  Large dataset performance is P2 (acceptable risk)
```

---

## Common Pitfalls to Avoid

1. **Vague scenarios**: "User exports tasks" vs "User exports 3 specific tasks and verifies CSV contains all data"
2. **Missing error paths**: Only testing happy path
3. **No priority distinction**: Everything marked P0
4. **Skipping Playwright hints**: Makes automation harder later
5. **Over-testing low-risk features**: 50 test scenarios for a simple form
6. **Under-testing high-risk features**: Skipping security tests on auth flows
7. **Incomplete execution mockup**: Missing edge cases that show up in testing
8. **No methodology justification**: Unclear why certain tests were chosen

---

## Integration with Workflow Pipeline

```
/brainstorm → /breakdown → /spec → /test-reqs → /decompose
                   ↓          ↓         ↓
              Layer 3     Spec    Test Requirements
              Blueprint           (Execution Mockup + Scenarios)
```

**How they connect**:
- `/breakdown` Layer 3 provides the technical blueprint
- `/spec` formalizes the specification
- `/test-reqs` takes that spec and generates test requirements
- `/decompose` uses spec + test requirements to create executable tasks

---

## Usage

**Starting the skill**:
- User says: **"/test-reqs"** or **"Generate test requirements for [feature]"**
- Provide the specification (from specification, breakdown output, user story, API doc)

**What you'll get**:
- Phase 1: Complete execution mockup (how the feature behaves)
- Phase 2: Test scenarios organized by methodology with BDD format
- Coverage summary showing test distribution

**Next steps**:
- Use execution mockup for documentation and understanding
- Use test scenarios to implement actual tests (Playwright, API tests, unit tests)
- Filter by priority (implement P0 first, then P1, P2 optional)
