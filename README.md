🧠 OWL → Wikibase Importer

A Python-based pipeline for transforming OWL ontologies into structured Wikibase knowledge graphs.

Overview

This repository provides a complete importer that reads an OWL/RDF ontology and maps its classes, individuals, and selected object properties into a Wikibase instance.
It preserves the semantic structure of the source ontology, ensures duplicate-free item creation, and enriches Wikibase with labels, aliases, descriptions, and external identifiers.

The importer is suitable for research knowledge graphs, semantic web projects, scientific knowledge bases, and environments where ontologies must be transformed into the Wikibase data model.

✨ Features
Class Import & Hierarchy Reconstruction

Detects all OWL (owl:Class) and RDFS (rdfs:Class) classes.

Creates or reuses Wikibase items for each class.

Anchors owl:Thing as the root class.

Inserts subclass_of (P18) statements to express the class hierarchy.

Individual Import

Detects all owl:NamedIndividual resources.

Creates or updates Wikibase items with:

rdfs:label / skos:prefLabel

skos:altLabel aliases

descriptions (rdfs:comment)

Adds instance_of (P16) statements linking individuals to their OWL classes.

Deduplication via Ontology IRIs

Each OWL resource retains its original IRI via the Wikibase property:
