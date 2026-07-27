# data-domain-01 — Context

Ubiquitous language for this domain: business terms, entities, source-system quirks, and
naming conventions the agents rely on when building, fixing, or advising on this domain's
data products.

## Language

**Salesforce Objects**: Key CRM entities including Opportunity, Account, Contact, User, and Stage. Critical for understanding the source data structure and dependencies when building ingestion pipelines.

**Medallion Layers**: Architectural pattern organizing the data warehouse: Bronze (raw ingested data), Silver (cleaned and deduplicated), Gold (business-ready facts and dimensions). This domain uses this structure consistently.

<!--
Append one entry per term, as it surfaces (never pre-populate speculative terms):

**<Term>**:
One or two sentences — what it means in this domain, and why it matters to a data product.
_Avoid_: near-synonyms this domain has rejected, and why.
-->
