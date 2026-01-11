# AI Planning Workflow

A structured workflow of Claude Code skills for transforming ideas into executable tasks through progressive refinement.

## Overview

This workflow helps you go from initial idea exploration to concrete, executable tasks with comprehensive test requirements. It's designed for AI-assisted development, where each step refines understanding and adds detail.

## The Complete Pipeline

```
/brainstorm → /breakdown → /spec → /test-reqs → /decompose
     ↓             ↓          ↓         ↓            ↓
  Explore       Layers    Formal    Test Reqs   Feature Areas
   Ideas                   Spec                  + Executable Tasks
```

## Skills

### 1. `/brainstorm` - Collaborative Solution Exploration

**Purpose**: Explore the solution space with diverse ideas until you find an approach worth building.

**Key Features**:
- User-directed iteration (you steer the exploration)
- Web research for existing solutions
- Pros/cons analysis
- Nested brainstorming (dive deep into specific ideas)
- Phase-by-phase execution (context → research → ideas → iteration → handoff)

**When to use**: When you need to explore options, aren't sure what to build, or want to validate ideas.

**Output**: Refined solution approach ready for breakdown.

---

### 2. `/breakdown` - Progressive Understanding Through Layers

**Purpose**: Transform complex ideas into three progressive layers of understanding.

**Key Features**:
- Layer 1: Narrative simplicity (analogy-based, no jargon)
- Layer 2: Structured detail (technical terms with definitions)
- Layer 3: Complete blueprint (implementation-ready)
- Diagrams at each level
- Shareable or teaching format

**When to use**: When you've picked an approach and want progressive understanding before implementing.

**Output**: Three-layer understanding with diagrams (Layer 3 feeds into `/spec`).

---

### 3. `/spec` - High-Level Specification Creation

**Purpose**: Create detailed, behavior-focused specifications describing WHAT needs to exist and WHY, not HOW.

**Key Features**:
- Structured template (Overview, Goals, Success Criteria, Features, etc.)
- Behavior over implementation
- Edge cases and constraints
- Out of scope section (prevent scope creep)
- Dependencies clearly stated

**When to use**: After breakdown, when you need a formal specification for a feature or system.

**Output**: Complete specification document ready for test requirements generation.

---

### 4. `/test-reqs` - Comprehensive Test Requirements

**Purpose**: Generate comprehensive test requirements through execution mockup + test scenarios.

**Key Features**:
- Phase 1: Execution mockup (behavioral walkthrough - happy path, error scenarios, edge cases)
- Phase 2: Test scenarios (BDD format with intelligent methodology selection)
- Priority levels (P0/P1/P2)
- Playwright automation hints for UI tests
- Coverage summary

**When to use**: When you have a spec and need test requirements.

**Output**: Execution mockup + test scenarios that feed into `/decompose`.

---

### 5. `/decompose` - Spec to Executable Tasks

**Purpose**: Transform specifications and test requirements into concrete, executable tasks organized by feature areas.

**Key Features**:
- 6-component hybrid analysis (spec analysis, test coverage, task boundary inference, etc.)
- Tasks sized for capable AI agents (respect agent potential)
- Natural language but specific test requirements
- Feature-level review tasks for integration verification
- Auto-selected tags and dependencies

**When to use**: When you have spec + test-reqs and need executable task breakdown.

**Output**: JSON with feature areas, tasks, test requirements, dependencies - ready for project planning.

---

## Workflow Usage

### Full Pipeline Usage

1. **Start with brainstorm** if you need to explore options:
   ```
   User: "/brainstorm ways to add user authentication"
   → Explores OAuth, email/password, magic links, etc.
   → User picks: "Email/password with session management"
   ```

2. **Breakdown the chosen approach**:
   ```
   User: "/breakdown email/password authentication with sessions"
   → Layer 1: Analogy (like a club bouncer checking ID and giving wristbands)
   → Layer 2: Components (login endpoint, session store, middleware)
   → Layer 3: Complete blueprint (API contracts, data schemas, flows)
   ```

3. **Create formal specification**:
   ```
   User: "/spec"
   [Provide breakdown Layer 3 output]
   → Structured spec with features, edge cases, success criteria
   ```

4. **Generate test requirements**:
   ```
   User: "/test-reqs"
   [Provide spec output]
   → Execution mockup (how it behaves)
   → Test scenarios (BDD format with priorities)
   ```

5. **Decompose into executable tasks**:
   ```
   User: "/decompose"
   [Provide spec + test-reqs output]
   → Feature areas with concrete tasks
   → Test requirements per task
   → Review tasks for integration verification
   ```

### Standalone Usage

Each skill can also be used independently:

- **Just brainstorming**: Explore ideas without committing to full workflow
- **Just breakdown**: Get progressive understanding of any concept
- **Just spec**: Formalize an existing idea into structured spec
- **Just test-reqs**: Generate tests for existing feature
- **Just decompose**: Break down existing spec into tasks

## Installation

### For Claude Code CLI

1. Copy the `skills/` directory to your project:
   ```bash
   cp -r skills /path/to/your/project/.claude/skills
   ```

2. Skills will be automatically available in Claude Code

### For Other AI Tools

The skills are written in markdown with clear phase-by-phase instructions. You can:
- Use them as prompts in any AI chat interface
- Adapt the templates for your tooling
- Extract specific phases you need

## Skill Format

Each skill follows this structure:

```markdown
---
name: skill-name
description: Brief description
model: claude-sonnet-4-20250514
---

# Skill Title

## Workflow Context
[Where it fits in pipeline, standalone usage]

## Execution Rhythm
[One phase per turn vs all at once]

## Core Principles
[Key principles guiding the skill]

## The Process
[Detailed phase-by-phase instructions]

## Examples
[Concrete examples]

## Tips
[Best practices and common pitfalls]
```

## Key Concepts

### Execution Rhythm

**Skills use different execution patterns**:

- **`/brainstorm`**: One phase per turn (allows course-correction)
- **`/breakdown`**: All layers at once (format provides progressive disclosure)
- **`/spec`**: All sections at once (template is self-organizing)
- **`/test-reqs`**: Both phases in one turn (mockup + scenarios)
- **`/decompose`**: One phase per turn (allows refinement)

### Task Sizing for AI Agents

Tasks are sized assuming **capable AI agents** (like full Claude chat sessions):
- **15 min**: Simple changes (add field, config tweak)
- **30 min**: Standard tasks (create endpoint, component) - most common
- **45 min**: Complex tasks (multi-step logic, integration)

**Don't over-split** - micromanaging reduces agent effectiveness.

### Test Requirements

Test requirements are **natural language but specific**:
- ✅ Good: "POST /api/login returns 200 with session token in HTTP-only cookie"
- ❌ Bad: "Login works correctly"

This specificity enables automated testing rigs while remaining human-readable.

### Review Tasks

Only **review tasks** are marked `[testing]`:
- Implementation tasks have test requirements (for automated verification)
- Review tasks verify feature-level integration
- Review happens before features combine at project level

## Examples

See the `skills/*/examples.md` files for detailed examples of each skill in action.

## Contributing

Contributions welcome! To add or improve skills:

1. Follow the skill format structure
2. Include workflow context
3. Provide concrete examples
4. Document execution rhythm
5. Add tips and common pitfalls

## License

MIT License - feel free to use, modify, and share.

## Credits

Created as part of the AI-assisted development workflow. Designed for use with Claude Code but adaptable to any AI development tool.

---

## Quick Start

**Have an idea?** Start with `/brainstorm`:
```
"Brainstorm ways to add real-time collaboration to my app"
```

**Know what to build?** Skip to `/breakdown`:
```
"Breakdown: WebSocket-based real-time collaboration with presence indicators"
```

**Have a spec?** Jump to `/test-reqs`:
```
"/test-reqs [paste your spec]"
```

**Ready to execute?** Use `/decompose`:
```
"/decompose [paste spec + test-reqs]"
```

The workflow is **flexible** - use what you need, skip what you don't.
