---
name: breakdown
description: Decomposes complex concepts, plans, architectures, or proposals into three progressive layers of understanding with diagrams at each level. Use when user asks to "break down", "break down in layers", "explain at different levels", or wants to understand something progressively from simple to detailed.
model: claude-sonnet-4-20250514
---

# Breakdown: Progressive Understanding Through Layers

This skill transforms complex ideas into three progressive layers of understanding, each with an accompanying diagram. Think of it as a telescope with three zoom levels: constellation → star cluster → individual stars.

## Workflow Context

**This skill can be used:**
- **As part of the planning pipeline**: `/brainstorm` → `/refine` → `/breakdown` → `/spec` → `/test-reqs` → `/decompose`
- **Standalone**: When you need progressive understanding of any concept (technical, business, abstract)

**When in the pipeline**: Takes output from `/brainstorm` and `/refine` (scope document) to create Layer 3 (blueprint) that respects defined boundaries and constraints. Feeds into `/spec`.

## Execution Rhythm: Generate All Layers at Once

**Unlike other skills**, this one typically generates output in a single turn (all 3 layers or the requested layer). The layered format itself provides progressive disclosure, so there's no need to pause between layers.

**User can request**:
- All layers (default): "Breakdown OAuth" → Generates Layer 1, 2, 3 in one response
- Single layer: "Give me a simple explanation of Docker" → Generates Layer 1 only
- Specific format: "Breakdown microservices in teaching format" → All layers with explicit labels

## Core Principle

**Progressive Disclosure**: Build understanding in stages, each layer adding precision while maintaining comprehension. Never jump to technical details before establishing intuition.

## Layer Selection & Format Logic

**IMPORTANT**: Before generating output, detect which layer(s) the user wants AND which format:

### Layer Selection

**Single Layer Requests** (output ONLY that layer):
- **Layer 1 triggers**: "simple", "basic", "ELI5", "quick", "overview", "high-level", "gist", "just the story", "analogy"
- **Layer 2 triggers**: "medium", "moderate", "detailed", "intermediate", "structured", "architecture view"
- **Layer 3 triggers**: "in-depth", "complete", "full", "comprehensive", "deep", "thorough", "blueprint", "implementation details", "all the details"

**All Layers** (default):
- Generic requests: "decompose", "break down", "explain", "how does X work" (without level qualifiers)
- Explicit all-layers: "all layers", "progressive", "layered explanation"

### Format Selection

**Shareable Format** (default - use natural headings):
- Most requests should use this format
- Headings are contextual and natural, not "Layer 1/2/3"
- Ready to post/share without looking like a learning framework

**Teaching Format** (explicit layer labels):
- Triggers: "teaching format", "show layers", "label the layers", "learning mode"
- Use explicit "Layer 1:", "Layer 2:", "Layer 3:" headings
- Good for explaining the decomposition methodology itself

**Examples**:
- "Give me a simple breakdown of Docker" → Layer 1 only, shareable format
- "Decompose microservices" → All 3 layers, shareable format
- "Decompose OAuth in teaching format" → All 3 layers, with "Layer X:" labels
- "Explain how Git works, show layers" → All 3 layers, with "Layer X:" labels

## The Three-Layer Structure

### Layer 1: Narrative Simplicity
**Goal**: "I get the gist" - Build intuition without cognitive overhead

**Format**:
- Pure narrative flow (NO bullet points)
- Strong analogy mapping complex concept to familiar domain
- Focus on WHAT and WHY, not HOW
- 3-5 paragraphs maximum
- End with simple diagram (3-5 elements max)

**Analogy Selection Criteria**:
- Choose familiar domains: cooking, traveling, building, ecosystems, mail delivery, orchestras
- Map 1:1: each key concept must have an analogy counterpart
- Maintain consistency: don't switch analogies mid-layer
- Test: Could a smart teenager understand this?

**Diagram Requirements**:
- ASCII art or simple mermaid flowchart
- 3-5 boxes/nodes maximum
- Uni-directional arrows preferred
- Labels in plain English (no jargon)
- Shows main flow or core relationship

**Example Analogy Mapping**:
```
API Gateway → Restaurant Host (routes guests to tables)
Microservice → Kitchen Station (specialized, independent)
Database → Pantry (stores ingredients)
```

---

### Layer 2: Structured Detail
**Goal**: "I could explain this to someone else" - Add precision while maintaining comprehension

**Format**:
- Opening narrative (2-3 sentences) bridging from Layer 1
- 3-5 categorized sections with descriptive headers
- 3-7 bullet points per section
- Introduce technical terms WITH brief inline definitions
- Mix of "what" and "why" in bullets
- End with structured diagram (8-12 elements)

**Section Organization**:
- Group related concepts logically
- Headers should be specific, not generic (e.g., "Request Flow & Routing" not "Components")
- Each section builds on the previous

**Terminology Introduction**:
- Define terms inline on first use: "...uses a load balancer (traffic distributor) to..."
- Link back to Layer 1 analogy when helpful
- No unexplained acronyms

**Diagram Requirements**:
- Show major subsystems or workflow stages
- Add swim lanes or grouping boxes
- Label data/control flows
- Show key decision points (diamonds in flowcharts)
- 8-12 elements typical

---

### Layer 3: Complete Blueprint
**Goal**: "I could build/execute this" - Expose every detail needed for implementation

**Format**:
- Comprehensive sections covering every component
- Nested bullets (multi-level detail)
- Technical specifics appropriate to domain:
  - **Code/Architecture**: API contracts, data schemas, file paths, dependencies
  - **Plans/Projects**: Timeline dependencies, resource requirements, risks
  - **Concepts/Ideas**: Edge cases, counterexamples, research citations
- Explicit tradeoffs and alternatives considered
- End with complete diagram (15-30 elements)

**Detail Requirements**:
- Error scenarios and handling
- Edge cases and limitations
- Performance characteristics or constraints
- Integration points with external systems
- Security considerations (if applicable)
- Rollback or failure strategies (if applicable)

**Diagram Requirements**:
- All components and relationships
- External dependencies clearly marked (dashed lines or different style)
- Data flows with types/formats labeled
- Bidirectional relationships where applicable
- Legend if using different shapes/styles
- 15-30 elements typical

---

## Execution Process

1. **Analyze Input**:
   - Identify domain (technical, business, abstract concept)
   - Extract core components to track through layers
   - Determine appropriate analogy domain for Layer 1

2. **Generate Layer 1**:
   - Choose analogy and map all key concepts
   - Write narrative flow (no bullets)
   - Create simple diagram
   - Verify: Is this jargon-free? Would it make sense to someone unfamiliar?

3. **Generate Layer 2**:
   - Write 2-3 sentence bridge from Layer 1
   - Organize into logical sections
   - Add bullets with technical terms defined inline
   - Create structured diagram showing flow/groupings
   - Verify: Does each new term get defined? Does this build smoothly from Layer 1?

4. **Generate Layer 3**:
   - Add comprehensive technical detail
   - Use nested bullets for complexity
   - Include all dependencies, edge cases, tradeoffs
   - Create complete system diagram
   - Verify: Could someone actually implement/execute this from this info?

5. **Quality Check**:
   - Analogy coherence (Layer 1)
   - Terminology progression (Layer 2 bridges properly)
   - Completeness (Layer 3 is executable)
   - Diagram progression (each adds appropriate detail)

---

## Diagram Syntax Examples

### Simple Flow (Layer 1)
```
┌─────────┐
│  Input  │
└────┬────┘
     │
     ▼
┌─────────┐
│ Process │
└────┬────┘
     │
     ▼
┌─────────┐
│ Output  │
└─────────┘
```

### Structured Flow with Branches (Layer 2)
```
┌──────────────────────┐
│   Input Handler      │
└──────┬───────────────┘
       │
   ┌───┴────┐
   │  Valid?│ (decision point)
   └───┬────┘
       │
    ┌──┴──┐
    ▼     ▼
┌────────┐ ┌────────┐
│Success │ │ Error  │
│Path    │ │Handler │
└───┬────┘ └───┬────┘
    │          │
    └────┬─────┘
         ▼
    ┌─────────┐
    │ Output  │
    └─────────┘
```

### Complex System (Layer 3)
```
┌─────────────────────────────────────┐
│        External Systems             │
│  ┌──────┐    ┌──────┐              │
│  │Auth  │    │ DB   │              │
│  │API   │    │      │              │
│  └──┬───┘    └───┬──┘              │
└─────┼───────────┼──────────────────┘
      │           │
      │  ┌────────┼────────┐
      ▼  ▼        ▼        ▼
   ┌──────┐   ┌──────┐ ┌──────┐
   │Gate  │───│Core  │─│Cache │
   │way   │   │Logic │ │Layer │
   └──┬───┘   └───┬──┘ └──────┘
      │           │
      │     ┌─────┴─────┐
      ▼     ▼           ▼
   ┌──────┐ ┌──────┐ ┌──────┐
   │UI    │ │API   │ │Queue │
   │Layer │ │Route │ │Proc  │
   └──────┘ └──────┘ └──────┘
```

---

## Domain-Specific Adaptations

### For Technical Architecture
- Layer 1: Use restaurant, factory, or mail system analogies
- Layer 2: Focus on data flow and component responsibilities
- Layer 3: Include API contracts, error codes, scaling considerations

### For Business Plans
- Layer 1: Use journey or ecosystem analogies
- Layer 2: Organize by business functions (marketing, ops, finance)
- Layer 3: Include resource requirements, timeline dependencies, risk mitigation

### For Abstract Concepts
- Layer 1: Use physical world analogies (gravity, ecosystems, construction)
- Layer 2: Introduce academic/formal terminology
- Layer 3: Include edge cases, counterexamples, research context

---

## Output Templates

### Shareable Format (DEFAULT)

Use natural, contextual headings that flow well when shared:

**All Three Layers**:
```markdown
# [Concept Name]

## The Essence
[or: Overview, Introduction, What Is It?]

[Pure narrative with analogy, 3-5 paragraphs, no bullets]

[Simple diagram]

---

## How It Works
[or: Architecture, Key Components, The System, Under the Hood]

**Big Picture**: [2-3 sentence bridge]

**[Section 1 Name]**:
- [Bullets with inline definitions]
...

[Structured diagram with 8-12 elements]

---

## Deep Dive
[or: Implementation Details, Complete Technical Breakdown, Full Architecture, The Blueprint]

### [Component/Aspect 1]
**Purpose**: [What this does and why it matters]

- [Technical details with nested bullets]
...

[Complete diagram with 15-30 elements]
```

**Single Layer** (choose appropriate heading):
- Layer 1 → "Overview", "The Essence", "Introduction", "What Is [Concept]?"
- Layer 2 → "How It Works", "Architecture", "Key Components", "The System"
- Layer 3 → "Deep Dive", "Implementation Details", "Complete Breakdown", "Technical Architecture"

### Teaching Format (Explicit Layers)

Only use when user explicitly requests "teaching format" or "show layers":

```markdown
# [Concept Name]

## Layer 1: The Simple Story

[Content...]

---

## Layer 2: The Architecture

[Content...]

---

## Layer 3: The Full Blueprint

[Content...]
```

**Default to shareable format** unless teaching format is explicitly requested.

## Common Pitfalls to Avoid

1. **Using jargon in Layer 1** - Ruins intuition building
2. **Skipping the narrative bridge in Layer 2** - Creates jarring transition
3. **Incomplete Layer 3** - If someone can't execute from it, add more detail
4. **Inconsistent analogies** - Confuses rather than clarifies
5. **Diagrams that don't match layer complexity** - Layer 1 should be SIMPLE

---

## Example Invocations

**Shareable format** (natural headings - DEFAULT):
- "Decompose how OAuth works" → All 3 layers, natural headings
- "Give me a simple explanation of Docker" → Layer 1 only, natural heading
- "I need a detailed breakdown of REST APIs" → Layer 2 only, natural heading
- "Show me the complete architecture of Kubernetes" → Layer 3 only, natural heading

**Teaching format** (explicit "Layer X:" labels):
- "Decompose OAuth in teaching format" → All 3 layers with "Layer 1/2/3" labels
- "Explain microservices, show layers" → All 3 layers with explicit labels
- "Break down this architecture in learning mode" → All 3 layers, teaching format

**Flexibility**:
- The skill adapts heading names to context (technical vs business vs abstract)
- You can mix depth and format: "Simple explanation of OAuth in teaching format"
- Default is always shareable unless you explicitly ask for teaching format
