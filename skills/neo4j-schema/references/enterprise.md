# Enterprise & Business Knowledge Graph — Domain Reference

Use this file when the user's graph involves organizations, people, products, deals, or business processes.

---

## Common Node Labels

### Person
Core entity for any individual — employee, customer, contact, executive.

| Property      | Type     | Notes                                      |
|---------------|----------|--------------------------------------------|
| id            | String   | UUID                                       |
| name          | String   | Full name                                  |
| email         | String   | Unique within domain                       |
| title         | String   | Job title                                  |
| department    | String   | Org unit (or use relationship to Dept node)|
| linkedinUrl   | String   | External profile                           |
| createdAt     | DateTime |                                            |

### Organization
Companies, agencies, subsidiaries, nonprofits.

| Property       | Type     | Notes                          |
|----------------|----------|--------------------------------|
| id             | String   | UUID                           |
| name           | String   | Legal or common name           |
| type           | String   | "company", "nonprofit", "govt" |
| domain         | String   | e.g., "acme.com"               |
| industry       | String   | Sector classification          |
| country        | String   | ISO 3166 country code          |
| foundedYear    | Integer  |                                |
| createdAt      | DateTime |                                |

### Product
Physical goods, software, services offered by an organization.

| Property      | Type     | Notes                          |
|---------------|----------|--------------------------------|
| id            | String   | UUID or SKU                    |
| name          | String   |                                |
| category      | String   |                                |
| description   | String   |                                |
| status        | String   | "active", "deprecated", etc.   |
| createdAt     | DateTime |                                |

### Deal / Opportunity
Sales deals, contracts, partnerships.

| Property      | Type     | Notes                              |
|---------------|----------|------------------------------------|
| id            | String   | UUID or CRM ID                     |
| name          | String   | Deal name                          |
| value         | Float    | Monetary value                     |
| currency      | String   | ISO 4217 (e.g., "USD")             |
| stage         | String   | "prospect", "negotiation", "closed"|
| closedAt      | Date     | Actual or expected close date      |
| createdAt     | DateTime |                                    |

### Location
Physical addresses, offices, regions.

| Property   | Type   | Notes               |
|------------|--------|---------------------|
| id         | String | UUID                |
| name       | String | Office or city name |
| city       | String |                     |
| country    | String | ISO 3166            |
| lat        | Float  | Latitude            |
| lon        | Float  | Longitude           |

### Event
Meetings, conferences, product launches, incidents.

| Property    | Type     | Notes                             |
|-------------|----------|-----------------------------------|
| id          | String   | UUID                              |
| name        | String   |                                   |
| type        | String   | "meeting", "conference", "launch" |
| startDate   | DateTime |                                   |
| endDate     | DateTime |                                   |
| description | String   |                                   |

### Tag / Category
Free-form labels for classification — skills, interests, industries.

| Property | Type   | Notes         |
|----------|--------|---------------|
| id       | String | UUID or slug  |
| name     | String | Display name  |
| type     | String | Tag namespace |

---

## Common Relationship Types

| Relationship       | From         | To           | Key Properties             |
|--------------------|--------------|--------------|----------------------------|
| WORKS_AT           | Person       | Organization | since, role, isCurrent     |
| MANAGES            | Person       | Person       | since                      |
| KNOWS              | Person       | Person       | strength (0.0–1.0), since  |
| FOUNDED            | Person       | Organization | year                       |
| OWNS               | Person/Org   | Organization | sharePercent               |
| SUBSIDIARY_OF      | Organization | Organization | since                      |
| PARTNER_WITH       | Organization | Organization | type, since                |
| SELLS              | Organization | Product      | since                      |
| INVOLVED_IN        | Person/Org   | Deal         | role ("buyer","seller")    |
| LOCATED_AT         | Person/Org   | Location     | isPrimary                  |
| ATTENDED           | Person       | Event        | role ("speaker","attendee")|
| HOSTED             | Organization | Event        |                            |
| TAGGED_WITH        | Any          | Tag          | confidence (if ML-assigned)|
| COMPETITOR_OF      | Organization | Organization | since                      |
| ACQUIRED           | Organization | Organization | date, value                |

---

## Recommended Constraints & Indexes

```cypher
-- Uniqueness constraints
CREATE CONSTRAINT person_id IF NOT EXISTS FOR (n:Person) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT org_id IF NOT EXISTS FOR (n:Organization) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT product_id IF NOT EXISTS FOR (n:Product) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT deal_id IF NOT EXISTS FOR (n:Deal) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT tag_name IF NOT EXISTS FOR (n:Tag) REQUIRE n.name IS UNIQUE;

-- Search indexes
CREATE TEXT INDEX person_name_text IF NOT EXISTS FOR (n:Person) ON (n.name);
CREATE TEXT INDEX org_name_text IF NOT EXISTS FOR (n:Organization) ON (n.name);
CREATE TEXT INDEX product_name_text IF NOT EXISTS FOR (n:Product) ON (n.name);

-- Full-text search across entity types
CREATE FULLTEXT INDEX enterprise_entity_search IF NOT EXISTS
FOR (n:Person|Organization|Product|Deal)
ON EACH [n.name, n.description, n.email];

-- Range index for filtering by date
CREATE RANGE INDEX deal_closedate IF NOT EXISTS FOR (n:Deal) ON (n.closedAt);
CREATE RANGE INDEX event_startdate IF NOT EXISTS FOR (n:Event) ON (n.startDate);
```

---

## Example Queries This Schema Enables

```cypher
-- Find all people who work at companies in the SaaS industry
MATCH (p:Person)-[:WORKS_AT]->(o:Organization {industry: "SaaS"})
RETURN p.name, o.name, p.title;

-- Find second-degree connections between two people
MATCH (a:Person {id: $id1})-[:KNOWS*2]-(b:Person {id: $id2})
RETURN DISTINCT b.name;

-- Org chart: find all reports under a manager
MATCH (manager:Person {id: $managerId})<-[:MANAGES*1..5]-(report:Person)
RETURN report.name, report.title;

-- Competitive landscape: org and its competitors
MATCH (o:Organization {id: $orgId})-[:COMPETITOR_OF]-(c:Organization)
RETURN c.name, c.industry, c.country;
```
