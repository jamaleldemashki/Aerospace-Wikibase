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
