---
name: neo4j-schema
description: >
  Design and generate Neo4j graph database schemas for knowledge graphs. Use this skill whenever
  the user wants to model data as a graph, define nodes and relationships for Neo4j, write Cypher
  schema statements (constraints, indexes), or create documentation for a knowledge graph. Trigger
  this skill for requests about graph data modeling, ontology design in Neo4j, knowledge graph
  architecture, or any task where the user needs to represent entities and their connections in
  a property graph. Also trigger when a user describes a domain (e.g., "I have companies, people,
  and documents that relate to each other") and wants help structuring it — even if they don't
  use the words "schema" or "Neo4j" explicitly.
---

# Neo4j Knowledge Graph Schema Skill

This skill guides the creation of high-quality, production-ready Neo4j graph database schemas for knowledge graphs. It covers beginner-friendly explanations, best-practice modeling patterns, Cypher schema statements, and schema documentation.

## Quick Concepts (for beginners)

Before diving in, here's the mental model:

- **Node**: A "thing" in your domain — a Person, Company, Document, Product. Nodes get **Labels** (like a category tag).
- **Relationship**: A connection between two nodes — `(:Person)-[:WORKS_AT]->(:Company)`. Relationships always have a direction and a **type** (written in UPPER_SNAKE_CASE by convention).
- **Property**: A key-value attribute on a node or relationship — `name: "Acme Corp"`, `since: 2019`.
- **Constraint**: A rule Neo4j enforces — e.g., every `:Company` must have a unique `id`.
- **Index**: A lookup structure to make queries faster — especially important for large graphs.

A knowledge graph is just a graph where the nodes and relationships carry semantic meaning — you're encoding *knowledge* about a domain, not just raw data.

---

## Workflow

Follow these steps in order:

### 1. Understand the Domain

Ask the user (or infer from context):
- What are the main **entities** (things) in their domain?
- What are the key **actions or connections** between them?
- What questions do they need to **answer with queries**? (This drives index design.)
- What domains apply? See `references/enterprise.md` or `references/documents.md` for domain starters.

If the user gives you a description of their data (even informal), extract the entities and relationships before proposing anything.

### 2. Design the Node Labels

For each entity type:
- Choose a **singular PascalCase label**: `Person`, `Company`, `Invoice`, `Article`
- Define its **properties** with names, types, and whether they're required
- Identify the **unique identifier** property (usually `id` or a domain key like `email`)
- Consider whether a node needs **multiple labels** (e.g., `:Person:Employee` or `:Document:Invoice`)

**Node property types in Neo4j:** `String`, `Integer`, `Float`, `Boolean`, `Date`, `DateTime`, `List`

### 3. Design the Relationship Types

For each connection:
- Use **UPPER_SNAKE_CASE verb phrases**: `WORKS_AT`, `AUTHORED`, `MENTIONS`, `PART_OF`
- Define **direction**: which node is the "subject"? Arrow points from subject to object.
- Add **properties on relationships** when the connection itself has attributes (e.g., `since`, `role`, `weight`)
- Prefer **specific relationship types** over generic ones: `MANAGES` beats `RELATES_TO`

**The reification rule**: If a relationship needs more than 2-3 properties, or if other nodes need to connect *to the relationship itself*, promote it to a node. For example, an `Employment` node between `Person` and `Company` can hold `startDate`, `endDate`, `title`, and link to a `Department`.

### 4. Write the Cypher Schema Statements

Output schema in this exact order:

#### a) Uniqueness Constraints (also create an index automatically)
```cypher
// Enforce unique IDs per node label
CREATE CONSTRAINT person_id_unique IF NOT EXISTS
FOR (n:Person) REQUIRE n.id IS UNIQUE;

CREATE CONSTRAINT company_id_unique IF NOT EXISTS
FOR (n:Company) REQUIRE n.id IS UNIQUE;
```

#### b) Existence Constraints (Neo4j Enterprise only — note this if used)
```cypher
// Required properties
CREATE CONSTRAINT person_name_exists IF NOT EXISTS
FOR (n:Person) REQUIRE n.name IS NOT NULL;
```

#### c) Additional Indexes (for frequently queried properties)
```cypher
// Text search index
CREATE TEXT INDEX person_name_text IF NOT EXISTS
FOR (n:Person) ON (n.name);

// Range index for date/number filtering
CREATE RANGE INDEX document_created_range IF NOT EXISTS
FOR (n:Document) ON (n.createdAt);

// Composite index for multi-property lookups
CREATE INDEX person_org_lookup IF NOT EXISTS
FOR (n:Person) ON (n.lastName, n.organizationId);
```

#### d) Full-text Indexes (for search across multiple properties/labels)
```cypher
CREATE FULLTEXT INDEX entity_search IF NOT EXISTS
FOR (n:Person|Company|Document) ON EACH [n.name, n.title, n.description];
```

### 5. Write Node & Relationship Definitions

Use this template for documentation:

```
## Node: Person
**Description**: An individual human in the knowledge graph.
**Labels**: :Person

| Property     | Type     | Required | Description                        |
|--------------|----------|----------|------------------------------------|
| id           | String   | ✅       | UUID, unique identifier            |
| name         | String   | ✅       | Full display name                  |
| email        | String   | ❌       | Primary email address              |
| createdAt    | DateTime | ✅       | When the node was added to graph   |

**Constraints**: `person_id_unique` (id must be unique)
**Indexes**: `person_name_text` (name, for search)

---

## Relationship: WORKS_AT
**Description**: Connects a Person to the Company they are employed by.
**Pattern**: `(:Person)-[:WORKS_AT]->(:Company)`

| Property   | Type   | Required | Description                        |
|------------|--------|----------|------------------------------------|
| since      | Date   | ❌       | Start date of employment           |
| role       | String | ❌       | Job title at this company          |
| isCurrent  | Boolean| ✅       | Whether this is their current role |
```

### 6. Produce the Final Output

Always output **three artifacts**:

**Artifact 1 — Schema Cypher** (`schema.cypher`)
All constraints and indexes as runnable Cypher, with comments. Safe to run multiple times (`IF NOT EXISTS`).

**Artifact 2 — Sample Data Cypher** (`sample_data.cypher`)  
5-10 `MERGE` or `CREATE` statements showing realistic example nodes and relationships. This helps users verify the schema looks right.

**Artifact 3 — Schema README** (`SCHEMA.md`)
A markdown document with:
- Overview of the knowledge graph's purpose
- Complete node definitions table
- Complete relationship definitions table  
- Index/constraint summary
- 3-5 example Cypher queries the schema enables

---

## Modeling Best Practices

**DO:**
- Use `MERGE` in sample data (not `CREATE`) so re-running doesn't duplicate data
- Give every node a UUID `id` property as its stable unique key
- Add `createdAt: datetime()` to all nodes for auditability
- Name relationships from the perspective of the "source" node reading naturally: `(person)-[:AUTHORED]->(doc)` reads as "person authored doc"
- Keep labels broad, use properties for subtypes: `:Document {type: "Invoice"}` rather than `:Invoice`... unless you query by subtype constantly
- Include a `source` property on nodes when data comes from multiple systems

**AVOID:**
- Generic relationship types like `RELATED_TO`, `HAS`, `IS` — be specific
- Storing arrays of IDs as properties — those should be relationships
- Deeply nested properties (use linked nodes instead)
- Using node labels as a replacement for properties when you have >20 label variants

---

## Domain Reference Files

For domain-specific node/relationship starters, read these files:

- **Enterprise & Business graphs** (orgs, people, products, deals): `references/enterprise.md`
- **Document & Content graphs** (articles, topics, authors, sources): `references/documents.md`

Read the relevant file(s) before proposing a schema if the user's domain fits one of these categories.

---

## Output Format Checklist

Before finishing, verify your output includes:
- [ ] All node labels defined with properties and types
- [ ] All relationship types defined with direction and properties
- [ ] `schema.cypher` with all constraints and indexes (using `IF NOT EXISTS`)
- [ ] `sample_data.cypher` with realistic MERGE examples
- [ ] `SCHEMA.md` documentation with tables and example queries
- [ ] Notes on any Neo4j Enterprise-only features used
- [ ] Beginner-friendly comments in all Cypher files
