# Document & Content Knowledge Graph — Domain Reference

Use this file when the user's graph involves documents, articles, research papers, web content, topics, authors, or media — or when the goal is to build a knowledge base or content intelligence system.

---

## Common Node Labels

### Document
The central entity — any piece of written content. Use the `type` property to distinguish subtypes.

| Property     | Type     | Notes                                                  |
|--------------|----------|--------------------------------------------------------|
| id           | String   | UUID or content hash                                   |
| title        | String   |                                                        |
| type         | String   | "article", "report", "email", "contract", "webpage"    |
| url          | String   | Source URL (if web content)                            |
| content      | String   | Full text (or store externally, keep summary here)     |
| summary      | String   | Short abstract                                         |
| language     | String   | ISO 639-1 code ("en", "fr")                            |
| publishedAt  | DateTime |                                                        |
| createdAt    | DateTime | When added to graph                                    |
| status       | String   | "draft", "published", "archived"                       |
| wordCount    | Integer  |                                                        |

### Author
A person or system that created a document. May also be a `:Person` node if you're building a combined enterprise+content graph.

| Property   | Type   | Notes                      |
|------------|--------|----------------------------|
| id         | String | UUID                       |
| name       | String |                            |
| email      | String |                            |
| affiliation| String | Organization name          |
| profileUrl | String |                            |

### Topic
A subject, concept, or theme that documents are about. The backbone of a knowledge graph — creates semantic bridges between documents.

| Property    | Type   | Notes                                     |
|-------------|--------|-------------------------------------------|
| id          | String | UUID or slug (e.g., "machine-learning")   |
| name        | String | Display name                              |
| description | String | Brief definition                          |
| parentTopic | String | (optional) id of broader parent topic     |
| aliases     | List   | Alternative names/synonyms                |

> Tip: Model topic hierarchies as relationships (`SUBTOPIC_OF`) rather than as a property — it makes traversal much easier.

### Source / Publisher
An organization or outlet that publishes content.

| Property  | Type   | Notes                     |
|-----------|--------|---------------------------|
| id        | String | UUID or domain            |
| name      | String |                           |
| domain    | String | e.g., "reuters.com"       |
| type      | String | "news", "academic", "blog"|
| country   | String | ISO 3166                  |

### Concept / Entity (Named Entity)
A real-world entity mentioned in documents — people, places, organizations, etc. extracted via NLP/NER.

| Property   | Type   | Notes                                       |
|------------|--------|---------------------------------------------|
| id         | String | UUID or canonical URI (e.g., Wikidata QID)  |
| name       | String | Canonical name                              |
| type       | String | "PERSON", "ORG", "LOCATION", "PRODUCT"      |
| wikidataId | String | External knowledge base link                |

### Claim / Fact
A specific assertion extracted from a document. Useful for fact-checking and knowledge extraction graphs.

| Property    | Type     | Notes                          |
|-------------|----------|--------------------------------|
| id          | String   | UUID                           |
| text        | String   | The claim as stated            |
| confidence  | Float    | 0.0–1.0 if ML-extracted        |
| extractedAt | DateTime |                                |

---

## Common Relationship Types

| Relationship       | From        | To          | Key Properties                    |
|--------------------|-------------|-------------|-----------------------------------|
| AUTHORED           | Author      | Document    | isPrimaryAuthor (Boolean)         |
| PUBLISHED_BY       | Document    | Source      |                                   |
| ABOUT              | Document    | Topic       | confidence, relevanceScore        |
| SUBTOPIC_OF        | Topic       | Topic       | (hierarchical — child to parent)  |
| MENTIONS           | Document    | Concept     | count, sentiment (-1 to 1)        |
| CITES              | Document    | Document    | context (excerpt of citation)     |
| SIMILAR_TO         | Document    | Document    | similarityScore (0.0–1.0), method |
| CONTAINS_CLAIM     | Document    | Claim       | excerpt (surrounding text)        |
| SUPPORTS           | Document    | Claim       | confidence                        |
| CONTRADICTS        | Document    | Claim       | confidence                        |
| TAGGED_WITH        | Document    | Topic/Tag   | assignedBy ("user","model")       |
| PREVIOUS_VERSION   | Document    | Document    | changedAt                         |
| TRANSLATED_FROM    | Document    | Document    | language                          |

---

## Recommended Constraints & Indexes

```cypher
-- Uniqueness constraints
CREATE CONSTRAINT document_id IF NOT EXISTS FOR (n:Document) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT author_id IF NOT EXISTS FOR (n:Author) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT topic_id IF NOT EXISTS FOR (n:Topic) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT topic_name IF NOT EXISTS FOR (n:Topic) REQUIRE n.name IS UNIQUE;
CREATE CONSTRAINT source_domain IF NOT EXISTS FOR (n:Source) REQUIRE n.domain IS UNIQUE;
CREATE CONSTRAINT concept_id IF NOT EXISTS FOR (n:Concept) REQUIRE n.id IS UNIQUE;

-- Text search on content fields
CREATE TEXT INDEX document_title_text IF NOT EXISTS FOR (n:Document) ON (n.title);
CREATE TEXT INDEX topic_name_text IF NOT EXISTS FOR (n:Topic) ON (n.name);

-- Date-range filtering
CREATE RANGE INDEX document_published_range IF NOT EXISTS FOR (n:Document) ON (n.publishedAt);

-- Full-text search across content
CREATE FULLTEXT INDEX content_search IF NOT EXISTS
FOR (n:Document|Claim)
ON EACH [n.title, n.summary, n.content, n.text];

-- Full-text for entity discovery
CREATE FULLTEXT INDEX entity_name_search IF NOT EXISTS
FOR (n:Concept|Author|Topic)
ON EACH [n.name, n.aliases];
```

---

## Example Queries This Schema Enables

```cypher
-- Find all documents about a topic, ranked by relevance
MATCH (d:Document)-[r:ABOUT]->(t:Topic {name: "Climate Change"})
RETURN d.title, d.publishedAt, r.relevanceScore
ORDER BY r.relevanceScore DESC;

-- Find topics connected through shared documents (topic co-occurrence)
MATCH (t1:Topic)<-[:ABOUT]-(d:Document)-[:ABOUT]->(t2:Topic)
WHERE t1.id < t2.id  -- avoid duplicates
RETURN t1.name, t2.name, count(d) AS sharedDocs
ORDER BY sharedDocs DESC LIMIT 20;

-- Trace a claim back to all supporting documents
MATCH (d:Document)-[:SUPPORTS]->(c:Claim {id: $claimId})
RETURN d.title, d.url, d.publishedAt
ORDER BY d.publishedAt DESC;

-- Find documents that cite a given document (forward citations)
MATCH (citing:Document)-[:CITES]->(source:Document {id: $docId})
RETURN citing.title, citing.publishedAt;

-- Navigate topic hierarchy: get all subtopics of "Technology"
MATCH (child:Topic)-[:SUBTOPIC_OF*1..3]->(parent:Topic {name: "Technology"})
RETURN child.name, count(*) AS depth
ORDER BY depth;

-- Find authors who write about overlapping topics
MATCH (a1:Author)-[:AUTHORED]->(:Document)-[:ABOUT]->(t:Topic)<-[:ABOUT]-(:Document)<-[:AUTHORED]-(a2:Author)
WHERE a1.id < a2.id
RETURN a1.name, a2.name, collect(DISTINCT t.name) AS sharedTopics;
```

---

## Tips for Document Knowledge Graphs

**On storing full text**: Neo4j isn't a document store — avoid putting large blobs in `content` if you have millions of documents. Instead, store a `summary` and a reference `url` or `storageKey`, and retrieve full text from S3/Elasticsearch when needed.

**On similarity edges**: Don't pre-compute `SIMILAR_TO` for all pairs — that's O(n²) edges. Instead, compute it on-demand with vector search or only for highly similar pairs above a threshold.

**On topic extraction**: If topics are ML-assigned, always store a `confidence` score on the `ABOUT` relationship and an `assignedBy: "model"` flag so you can filter or audit them later.

**On versioning**: Use `PREVIOUS_VERSION` relationships to build a document history chain. The "current" version is the one with no outgoing `PREVIOUS_VERSION`.
