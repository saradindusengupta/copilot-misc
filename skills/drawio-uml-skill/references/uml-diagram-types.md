# UML Diagram Types Reference

## Table of Contents
1. [Class Diagrams](#1-class-diagrams)
2. [Sequence Diagrams](#2-sequence-diagrams)
3. [Use Case Diagrams](#3-use-case-diagrams)
4. [Activity Diagrams](#4-activity-diagrams)
5. [State Machine Diagrams](#5-state-machine-diagrams)
6. [Component Diagrams](#6-component-diagrams)
7. [UML Multiplicity Quick Reference](#7-uml-multiplicity-quick-reference)
8. [Common Mistakes & Anti-patterns](#8-common-mistakes--anti-patterns)

---

## 1. Class Diagrams

### Purpose
Model the static structure of a system — classes, their attributes/methods, and the relationships between them.

### Elements

**Class**
```
┌────────────────┐
│   ClassName    │  ← name compartment (bold)
├────────────────┤
│ - attr: Type   │  ← attribute compartment
│ + attr2: Type  │
├────────────────┤
│ + method(): R  │  ← operation compartment
│ # helper(): void│
└────────────────┘
```

**Interface** — same three-compartment layout, with `«interface»` stereotype above the name, or the name in italics.

**Abstract class** — class name in *italics*, or `{abstract}` constraint.

**Enumeration** — `«enumeration»` stereotype; values in the attribute compartment.

### Relationships

| Relationship | Meaning | Direction | Notation |
|---|---|---|---|
| **Association** | "uses" or "knows about" | Undirected or directed | Solid line |
| **Directed association** | A references B | A → B | Solid line with open arrow |
| **Aggregation** | "has a" (weak ownership) | Whole ◇→ Part | Solid line, hollow diamond at whole |
| **Composition** | "consists of" (strong ownership) | Whole ◆→ Part | Solid line, filled diamond at whole |
| **Dependency** | "uses temporarily" | A - - → B | Dashed line, open arrow |
| **Generalization** | Inheritance | Child → Parent | Solid line, hollow triangle at parent |
| **Realization** | Implements interface | Class - - → Interface | Dashed line, hollow triangle at interface |
| **`«include»`** | Required inclusion | Base → Included | Dashed with `«include»` label |
| **`«extend»`** | Conditional extension | Extension → Base | Dashed with `«extend»` label |

### Multiplicity notation

Goes at both ends of an association line:
```
Customer ─────────── Order
         1        0..*
```
"One customer places zero or more orders."

### Validation rules (class diagrams)
1. Generalization arrows must point child → parent (not the other way)
2. Composition implies the part cannot exist without the whole — don't model a part as existing independently
3. Interfaces should have no attributes (only operations, or constants)
4. Abstract classes must have at least one abstract operation (or the abstraction adds no value)
5. Avoid duplicate attribute names within a class hierarchy (inherited vs. redefined must be intentional)
6. Multiplicity on both ends of every association is best practice

---

## 2. Sequence Diagrams

### Purpose
Model the dynamic interaction between objects over time, showing the order of messages exchanged.

### Elements

| Element | Description |
|---|---|
| **Actor** | External user or system (stick figure) |
| **Object/Participant** | Labeled box at the top: `objectName: ClassName` |
| **Lifeline** | Vertical dashed line below each participant |
| **Activation bar** | Narrow rectangle on a lifeline — object is "active" |
| **Message** | Horizontal arrow from sender to receiver |
| **Return message** | Dashed arrow back to caller |
| **Self-message** | Arrow looping back to same lifeline |
| **Combined fragment** | Box with `alt`, `opt`, `loop`, `par`, `ref` label |

### Message types

| Type | Notation | Meaning |
|---|---|---|
| Synchronous call | ──────► (filled arrowhead) | Caller waits for return |
| Asynchronous | ──────> (open arrowhead) | Caller does not wait |
| Return | ◄- - - - (dashed) | Response from callee |
| Create | ──────► to object box | Creates new object |
| Destroy | ──────► to X | Destroys object |

### Combined fragments

| Keyword | Meaning |
|---|---|
| `alt` | Alternatives (if/else) — operands separated by dashed line |
| `opt` | Optional (single guarded block) |
| `loop [min, max]` | Repetition |
| `par` | Parallel execution |
| `ref` | Reference to another sequence diagram |
| `break` | Exception / early exit |
| `critical` | Atomic / non-interruptible |

### Validation rules (sequence diagrams)
1. Synchronous messages must have a matching return message (dashed)
2. Activation bars open on message receipt and close on return send
3. `alt` fragments must have at least two operands; guards like `[condition]` on each
4. Object creation: the message arrow must point to the participant box, not the lifeline
5. Destruction: activation bar ends at the X mark; no further messages from that lifeline

---

## 3. Use Case Diagrams

### Purpose
Model system functionality from the user's perspective — what the system does, not how.

### Elements

| Element | Description |
|---|---|
| **Actor** | Role that interacts with the system (stick figure, outside boundary) |
| **Use case** | Oval/ellipse inside system boundary — a function the system performs |
| **System boundary** | Rectangle containing all use cases |
| **Association** | Solid line connecting actor to use case |
| **`«include»`** | Dashed arrow; base → included (always executed) |
| **`«extend»`** | Dashed arrow; extension → base (conditionally executed) |
| **Generalization** | Inheritance between actors or between use cases |

### `«include»` vs `«extend»`

```
PlaceOrder ──«include»──► ValidatePayment
                          (always runs when PlaceOrder runs)

PlaceOrder ◄──«extend»── ApplyDiscount
                          (optionally extends PlaceOrder)
```

- **`«include»`**: The included use case is *mandatory*. Arrow goes from base TO included.
- **`«extend»`**: The extending use case is *optional*. Arrow goes from extension TO base.

### Validation rules (use case diagrams)
1. Actors must be outside the system boundary
2. Use cases must be inside the system boundary
3. Every use case must be reachable from at least one actor (directly or via `«include»`)
4. `«extend»` relationships should specify an extension point on the base use case
5. Actors can be connected to actors only via generalization (not association)
6. Don't model steps or flows inside a use case — that's what sequence/activity diagrams are for

---

## 4. Activity Diagrams

### Purpose
Model workflows, business processes, or algorithm logic — the sequence of actions and decisions.

### Elements

| Element | Shape | Description |
|---|---|---|
| **Initial node** | ● (filled circle) | Start of flow |
| **Activity final** | ⊙ (circle with dot) | End of entire flow |
| **Flow final** | ✕ (circle with X) | Ends one flow branch only |
| **Action** | Rounded rectangle | A step/task |
| **Decision** | ◇ (diamond) | Branch — guards `[condition]` on each outgoing edge |
| **Merge** | ◇ (diamond) | Joins alternate branches |
| **Fork** | Thick horizontal bar | Splits into parallel flows |
| **Join** | Thick horizontal bar | Synchronizes parallel flows |
| **Swimlane** | Vertical/horizontal partition | Assigns responsibility to actor/role |

### Validation rules (activity diagrams)
1. Every path from initial node must eventually reach an activity final or flow final
2. Decision diamonds must have guards `[condition]` on all outgoing edges; guards should be mutually exclusive
3. Fork and join bars must be balanced (same number of flows in and out, collectively)
4. Swimlane boundaries must be consistent — an action belongs to exactly one lane
5. Avoid looping back to an initial node; use a `loop` edge back to the action itself

---

## 5. State Machine Diagrams

### Purpose
Model the lifecycle of an object — its states and the transitions triggered by events.

### Elements

| Element | Shape | Description |
|---|---|---|
| **Initial pseudostate** | ● | Starting point (not a real state) |
| **State** | Rounded rectangle | A condition the object can be in |
| **Final state** | ⊙ | Termination |
| **Transition** | Arrow | Labeled `event [guard] / action` |
| **Internal transition** | Listed inside state | Doesn't leave the state |
| **Composite state** | State containing sub-states | Nested state machine |
| **History pseudostate** | ⓗ | Resumes last active sub-state |
| **Junction** | ● (small filled circle) | Static conditional branch |
| **Choice** | ◇ | Dynamic conditional branch |

### Transition label syntax

```
EventName [guardCondition] / actionExpression
```
All parts are optional:
- `click / updateUI` — event triggers action, no guard
- `[balance < 0]` — guard-only (checked without event)
- `timeout [retries > 3] / logError` — full form

### Validation rules (state diagrams)
1. Must have exactly one initial pseudostate
2. Every state must be reachable from the initial pseudostate
3. Every non-final state must have at least one outgoing transition
4. Guards on transitions from the same source state should be mutually exclusive (or an `else` guard)
5. Don't confuse initial pseudostate (filled circle) with final state (circle-in-circle)

---

## 6. Component Diagrams

### Purpose
Model the high-level software architecture — components, interfaces they provide/require, and dependencies.

### Elements

| Element | Description |
|---|---|
| **Component** | Box with `«component»` stereotype or component icon |
| **Provided interface** | Lollipop symbol (circle on a stick) — what the component offers |
| **Required interface** | Socket symbol (half-circle) — what the component needs |
| **Port** | Small square on the component boundary |
| **Dependency** | Dashed arrow — one component depends on another |
| **Connector** | Joins a required interface to a provided interface |
| **Subsystem** | Large component containing sub-components |

### Validation rules (component diagrams)
1. Every required interface should connect to a provided interface
2. No circular dependencies between top-level components (warn if present)
3. Internal structure (sub-components) should only be shown when detail is necessary

---

## 7. UML Multiplicity Quick Reference

| Notation | Meaning |
|---|---|
| `1` | Exactly one |
| `0..1` | Zero or one (optional) |
| `*` or `0..*` | Zero or more |
| `1..*` | One or more |
| `2..5` | Between 2 and 5 |
| `n` | Exactly n |

---

## 8. Common Mistakes & Anti-patterns

| Mistake | Correct approach |
|---|---|
| Generalization arrow pointing parent → child | Arrow always points child → parent |
| Using composition when parts can exist independently | Use aggregation (hollow diamond) instead |
| Missing multiplicity on associations | Add multiplicity to both ends |
| Use case diagram showing internal steps | Use sequence or activity diagram for flows |
| Sequence diagram without return messages for sync calls | Add dashed return arrows |
| Activity diagram with dead-end paths | All paths must reach a final node |
| State diagram with unreachable states | Every state must be reachable |
| Interface with attributes | Interfaces have operations only |
| Actor inside system boundary | Actors are always outside |
