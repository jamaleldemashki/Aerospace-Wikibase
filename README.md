# Aerospace-Wikibase
This repository holds the Python pipeline + guidelines that mirror an OWL ontology into Wikibase instance.
## 1  What the pipeline does

| OWL construct        | Wikibase result                                                                                                                   |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Class**            | Item → *P1 «is ontology class»* → Q1                                                                                              |
| **Object property**  | Property (datatype **Wikibase Item**)                                                                                             |
| **Data property**    | Property (datatype **String**) *(none in v0)*                                                                                     |
| **Annotation prop.** | • built‑ins *(label / desc / alias)*• or extra property per mapping table                                                         |
| **Named individual** | Item with label / desc / alias + P1 → class + annotation claims (*source*, *wikidata uri*, *orkg id*) + **internal object links** |
| External IRIs        | Skipped for now *(future: stub items)*                                                                                            |
| `<rdf:Description>`  | Ignored for now                                                                                                                   |

---
