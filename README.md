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
| `<rdf:Description>`  | (Still not done)                                                                                                                  |

---
## 2  Environment secrets

Create a file called ``environment.env:

```properties
WIKIBASE_API_URL=https://everything4everyone.wikibase.cloud/w/api.php
WIKIBASE_SPARQL_URL=https://everything4everyone.wikibase.cloud/query/sparql
WIKIBASE_USER=<bot‑username>
WIKIBASE_PASSWORD=<bot‑password>
```

---
## 3  Globals you may tweak

```python
MEDIAWIKI_API_URL   = os.getenv("WIKIBASE_API_URL")
SPARQL_ENDPOINT_URL = os.getenv("WIKIBASE_SPARQL_URL")
ONTOLOGY_FILE       = "slr_reviewed.owl"

DRY_RUN             = True      # ← set False to write
SLEEP_BETWEEN_WRITES= 1.0       # polite throttle (sec)

ONTOLOGY_CLASS_QID  = "Q1"      # item = "Ontology class"
CLASS_MEMBERSHIP_PID= "P1"      # property = "instance of ontology class"
```
## Runtime tips
**Dry‑run workflow** : run once with `DRY_RUN=True` for a preview, then flip to `False` to commit.
- **Bot flag** → `WikibaseIntegrator(..., is_bot=True)` (higher maxlag).
- If you see `maxlag. sleeping …` the server is busy; the script auto‑retries—slow down via `SLEEP_BETWEEN_WRITES` if needed.
- Restart your Python session after changing **GLOBALS** or **ANNOTATION\_HANDLERS**.
