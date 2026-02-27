---
name: drawio-uml
description: >
  Read, parse, explain, validate, convert, and generate UML diagrams in draw.io format.
  Trigger this skill whenever the user shares draw.io XML, uploads a .drawio or .xml file,
  asks about a UML diagram, wants to convert a diagram to code or documentation, needs to
  validate UML correctness, or wants to generate a draw.io diagram from a description or
  natural language. Also trigger when the user pastes mxGraph XML, mentions "draw.io",
  "diagrams.net", or references class diagrams, sequence diagrams, use case diagrams,
  activity diagrams, or any UML construct — even if they don't use the word "UML" explicitly.
  This skill covers all four core workflows: parse/explain, convert to code/docs,
  validate structure, and generate from description.
---

# draw.io UML Skill

This skill enables Claude to work expertly with UML diagrams in draw.io format across four workflows:

1. **Parse & Explain** — Read draw.io XML and explain the diagram in plain English
2. **Convert** — Transform a diagram into code stubs, documentation, or other formats
3. **Validate** — Check UML correctness and flag structural issues
4. **Generate** — Create draw.io XML from natural language descriptions or code

Read the relevant reference file(s) before starting:
- `references/drawio-xml-format.md` — mxGraph XML structure, cell styles, shape names
- `references/uml-diagram-types.md` — UML semantics for all supported diagram types
- `references/generation-templates.md` — XML templates for generating diagrams

---

## Workflow Selection

Determine which workflow(s) apply based on the user's request:

| User says / does | Workflow |
|---|---|
| Pastes XML, uploads .drawio file, shares a diagram image | **Parse & Explain** |
| "Convert this to code", "generate docs", "turn this into Python" | **Convert** |
| "Is this correct?", "check my diagram", "what's wrong with this?" | **Validate** |
| "Create a diagram for...", "draw me a class diagram of..." | **Generate** |

Multiple workflows may apply — e.g., parse first, then validate, then convert.

---

## Workflow 1: Parse & Explain

### Input handling

**XML input (copy-pasted or file):**
1. Read `references/drawio-xml-format.md` to understand the mxGraph structure
2. Parse all `<mxCell>` elements — separate vertices (shapes) from edges (connections)
3. Identify diagram type from shape styles (see reference file)
4. Extract semantic UML elements

**Image input:**
1. Visually identify the diagram type
2. Extract all visible UML elements (classes, actors, lifelines, etc.)
3. Proceed with semantic understanding as if you had the XML

**Natural language description (no diagram yet):**
→ Switch to **Workflow 4: Generate**

### Parsing steps

1. **Identify diagram type** — class, sequence, use case, activity, state, component, deployment
2. **Extract elements** — nodes/vertices with their UML roles
3. **Extract relationships** — edges with labels, multiplicity, and arrow styles
4. **Build semantic model** — reconstruct the UML meaning

### Output format

Present the explanation as:
- **Diagram type** and overall purpose
- **Key elements** listed with their roles
- **Relationships** described in plain English (e.g., "Order *has many* OrderItems")
- **Summary** of what the diagram communicates architecturally

Always offer to convert to code or validate afterward.

---

## Workflow 2: Convert to Code / Documentation

### Supported output formats

| Target | Applicable diagram types |
|---|---|
| Python class stubs | Class diagrams |
| Java class stubs | Class diagrams |
| TypeScript interfaces | Class diagrams |
| SQL schema (DDL) | Class diagrams (entity-style) |
| Markdown documentation | Any diagram type |
| PlantUML | Any diagram type |
| Mermaid | Class, sequence, state, activity |
| OpenAPI stub | Class diagrams (service-style) |

### Conversion steps

1. Parse the diagram (Workflow 1 steps)
2. Confirm target language/format with the user if not specified
3. Map UML elements → target constructs:
   - Class → class/interface/table
   - Attributes → fields/columns with types (infer if not shown)
   - Operations/methods → function signatures
   - Associations → foreign keys / object references / imports
   - Inheritance → extends/implements/parent class
   - Multiplicity → cardinality constraints or collection types
4. Generate complete, syntactically correct output
5. Add docstrings/comments referencing the original diagram element names

### Type inference rules

When types are missing from the diagram:
- Attribute names ending in `_id`, `Id` → Integer or UUID
- Names like `name`, `title`, `description` → String
- Names like `created_at`, `updated_at` → DateTime
- Names like `is_*`, `has_*`, `enabled` → Boolean
- Names like `count`, `quantity`, `amount` → Integer or Float
- Always note inferred types with a comment: `# inferred`

---

## Workflow 3: Validate UML Structure

### Validation checklist by diagram type

Read `references/uml-diagram-types.md` for full UML rules. Key checks:

**Class diagrams:**
- [ ] Every class has a name
- [ ] Attributes have types (warn if missing)
- [ ] Inheritance arrows point from child → parent
- [ ] Multiplicity is present on associations (warn if missing)
- [ ] No circular inheritance
- [ ] Abstract classes/interfaces are marked
- [ ] Association class (if present) connects to the correct relationship

**Sequence diagrams:**
- [ ] Every lifeline has an actor or object label
- [ ] Messages flow left-to-right or right-to-left with direction
- [ ] Activation bars are properly opened and closed
- [ ] Return messages exist for synchronous calls
- [ ] `alt`/`opt`/`loop` fragments are labeled

**Use case diagrams:**
- [ ] System boundary is present
- [ ] All use cases are inside the system boundary
- [ ] Actors are outside the system boundary
- [ ] `<<include>>` and `<<extend>>` relationships are labeled
- [ ] Actors connect to use cases (not to other actors directly)

**Activity diagrams:**
- [ ] Has an initial node (filled circle)
- [ ] Has a final node (filled circle with ring)
- [ ] All branches (decision nodes) have labeled guards
- [ ] No dead ends (every path leads to the final node)
- [ ] Fork/join bars are balanced

**State diagrams:**
- [ ] Has an initial pseudostate
- [ ] Every transition is labeled with event [guard] / action
- [ ] No unreachable states
- [ ] Final state present if the machine terminates

### Validation output format

Report findings as:
- ✅ **Valid** — what is correct
- ⚠️ **Warning** — present but not best practice (e.g., missing multiplicity)
- ❌ **Error** — structurally incorrect per UML spec
- 💡 **Suggestion** — optional improvement

Always end with a summary: "X errors, Y warnings, Z suggestions."

---

## Workflow 4: Generate draw.io XML from Description

Read `references/generation-templates.md` for XML templates.

### Generation steps

1. **Clarify requirements** if ambiguous:
   - What type of diagram? (infer from context if possible — don't ask if obvious)
   - What domain/system is being modeled?
   - Any specific elements to include?

2. **Design the UML model** mentally first:
   - List all elements and their types
   - Define all relationships with direction and multiplicity
   - Apply UML conventions (see `references/uml-diagram-types.md`)

3. **Lay out the diagram** (positioning):
   - Use a grid: 200px spacing between elements horizontally, 150px vertically
   - Place parent classes above children
   - Place callers to the left of callees in sequence diagrams
   - Actors on left margin for use case diagrams

4. **Generate the XML** using templates from `references/generation-templates.md`:
   - Assign unique integer IDs starting from 2 (0 and 1 are reserved for root/default)
   - Wrap in the standard mxGraph envelope
   - Set `compressed="false"` so the XML is human-readable

5. **Validate your own output** (apply Workflow 3 checks)

6. **Deliver** the XML in a code block labeled `xml` and tell the user how to import it:
   > Copy the XML → Open draw.io → Extras → Edit Diagram → Paste → OK

---

## General Guidelines

- **Always read the relevant reference file(s)** before starting a task
- **Be explicit about assumptions** — if you infer a type, relationship direction, or missing label, say so
- **Ask at most one clarifying question** before proceeding — don't block on perfection
- **Offer next steps** — after parsing, offer to convert or validate; after generating, offer to validate
- **Handle compressed XML** — draw.io sometimes compresses the diagram data with base64+deflate; if you see `<mxGraphModel>` with a text content blob rather than child elements, note this and ask the user to export as uncompressed XML (File → Properties → uncheck "Compress XML")
