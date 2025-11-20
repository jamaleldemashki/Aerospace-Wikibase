# OWL → Wikibase Importer  
A Python-based pipeline for transforming OWL ontologies into structured Wikibase knowledge graphs.

## Overview

This repository provides an importer that reads an OWL/RDF ontology and maps its classes, individuals, and selected object properties into a Wikibase instance.  
It preserves the semantic structure of the source ontology, avoids duplicate items via ontology IRIs, and enriches Wikibase with labels, aliases, descriptions, and external identifiers.

The importer is suitable for research knowledge graphs, semantic web projects, scientific knowledge bases, and any setup where ontologies must be transformed into the Wikibase data model.

## Features

### Class import and hierarchy reconstruction

- Detects all OWL (`owl:Class`) and RDFS (`rdfs:Class`) classes.
- Creates or reuses Wikibase items for each class.
- Anchors `owl:Thing` as the root class.
- Inserts `subclass_of` statements (e.g. `P18`) to represent the class hierarchy.

### Individual import

- Detects all `owl:NamedIndividual` resources.
- Creates or updates Wikibase items using:
  - `rdfs:label` / `skos:prefLabel`
  - `skos:altLabel` aliases
  - `rdfs:comment` descriptions
- Adds `instance_of` statements (e.g. `P16`) linking individuals to their OWL classes.

### Deduplication via ontology IRIs

Each OWL resource keeps its original IRI via a dedicated Wikibase property, for example:

- `P17 = ontology_iri`

A SPARQL query checks if an item with the same ontology IRI already exists.  
If yes, the script reuses that QID.  
If not, it creates a new item and attaches the IRI.

### Property mapping

The importer never auto-creates properties.  
All properties must exist manually in the Wikibase instance and are mapped explicitly through two configuration layers:

1. A property ID mapping:

   ```python
   PROPS = {
       "instance_of": "P16",
       "ontology_iri": "P17",
       "subclass_of": "P18",
       "wikidata_uri": "P3",
       "source": "P2",
       "orkg_id": "P1",
       ...
   }
2. A predicate-to-property mapping for OWL object properties:
   ```python
    PREDICATE_TO_PROPSKEY = {
        # "has_data_model": "has_data_model",
        # "has_process": "has_process",
    }
Only predicates listed in PREDICATE_TO_PROPSKEY are turned into Wikibase statements.

### Annotation preservation
For each class or individual, the importer extracts and imports:
- Labels
- Description
- Aliases
- Wikidata IDs (QID or URL)
- ORKG IDs
- Source metadata

Aliases are inserted both as user-visible aliases and as machine-queryable statements.

### Throttled, robust Wikibase client
The included WBClient provides:
- Dynamic timeouts with exponential backoff
- Retry logic for transient HTTP/API errors
- A configurable delay (sleep_between) between requests to avoid overloading the server
- A full dry-run mode that prints intended API actions without executing them

## Requirements
Create a file called requirements.txt with:
  ```python
  rdflib>=6.3.0
  requests>=2.31.0
  tqdm>=4.66.0
  python-dotenv>=1.0.0
```
Install dependencies:
```python
  pip install -r requirements.txt
```
### Environment variables (bot.env)
The script loads configuration from a local bot.env file that is not part of the repository and must be created manually.
Example:
```python
  WB_API_URL=https://your-wikibase/api.php
WB_SPARQL_URL=https://your-wikibase/sparql
WB_USERNAME=YourBotUser
WB_PASSWORD=YourBotPassword
```
Notes:
- bot.env must not be committed to version control because it contains credentials.
- Each user must create their own bot.env for their Wikibase tenant.

## Adapting the importer to your Wikibase and OWL file
This importer is generic, but several parts must be tailored to your setup.

1. Update the property mapping (PROPS)
   Inside the script, adjust the PROPS dictionary so that its values match the actual P-IDs in your Wikibase instance:
   ```python
     PROPS = {
      "instance_of": "P16",
      "ontology_iri": "P17",
      "subclass_of": "P18",
      "wikidata_uri": "P3",
      "source": "P2",
      "orkg_id": "P1",
      ...
      }
   ```
  If your installation uses different P-IDs, change them accordingly.
2. Map OWL object properties (PREDICATE_TO_PROPSKEY)
Define which predicates from your ontology should be converted into Wikibase statements:
```python
  PREDICATE_TO_PROPSKEY = {
    # "has_data_model": "has_data_model",
    # "has_process":    "has_process",
}
```
Only these predicates are imported as statements.
This prevents accidental noise in your schema. 
3. Set the correct OWL file path
In your usage code or notebook, adjust the call to import_path:
```python
  from import_owl_to_wikibase import import_path
  
  rp = import_path("your_ontology.owl", dry_run=True)
```
Replace "your_ontology.owl" with the path to your OWL/RDF file (can also be a folder).
4. Tune request throttling 
To avoid overloading your Wikibase instance, use the configurable timeout and delay:
```python
  rp = import_path(
    "ontology.owl",
    dry_run=False,
    request_timeout=10.0,  # base timeout per request (seconds)
    sleep_between=0.2      # delay between API calls (seconds)
)
```
- Increase sleep_between for slower or shared servers.
- decrease it on a local test Wikibase if needed.

### Running the importer

#### Dry-run mode (recommended first step)
Dry-run mode performs all parsing and planning but does not write to Wikibase.
Write actions are printed instead.
```python
  from import_owl_to_wikibase import import_path

  rp = import_path("ontology.owl", dry_run=True)
  print(rp)
```
#### Full import
Once the configuration and dry-run look correct, run a real import:
```python
  from import_owl_to_wikibase import import_path

  rp = import_path(
      "ontology.owl",
      dry_run=False,
      request_timeout=10.0,
      sleep_between=0.2
  )
  print(rp)
```
#### Example report 
The importer returns a Report object summarizing its actions:
```python
  Report(
  files=['ontology.owl'],
  classes=120,
  subclass_links=350,
  individuals=89,
  instance_links=130,
  aliases=47,
  warnings=[...]
)
```
This helps track how many items and links were created, and lists any warnings (e.g. skipped entities or invalid IRIs).




