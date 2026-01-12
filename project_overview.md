# Project Overview

## Repositories

### Project Repository

- [dataesr/skolar](https://github.com/dataesr/skolar)

#### Current Pipeline (Paragraph Analysis)

- [skolar/project/server/main/pipeline.py](https://github.com/dataesr/skolar/blob/main/project/server/main/pipeline.py#L102)

Steps:
1. Download PDF
2. GROBID processing
3. Paragraph parsing:
   - XML paragraph extraction
   - Paragraph type detection
   - LLM agent call

### AI Pipeline Repository (Finetuning & Inference)

- [ml-hub/libs/ai_core/pipelines/pipelines](https://github.com/dataesr/ml-hub/blob/main/libs/ai_core/pipelines/pipelines)


## Data

### Corpus BSO 
Corpus split by year (2013-2025)
- `https://storage.gra.cloud.ovh.net/v1/AUTH_32c5d10cb0fe4519b957064a111717e3/bso_dump/bso-publications-latest_split_{year}_enriched.jsonl.gz`


### S3 Bucket
- `skolar`

Download script
- [notebooks/download_s3.ipynb](./notebooks/download_s3.ipynb)

Bucket file architecture:

- /llm
→ LLM results in .jsonl (raw + structured)

- /filter
→ paragraphs in .jsonl

- /pdf
→ PDF files

- /grobid
→ TEI XML (GROBID output)

- /publisher-xml
→ Publisher XML (non-GROBID)

### External Data Sources

SCANR organizations dump (+ wikidata)
- [https://scanr-data.s3.gra.io.cloud.ovh.net/production/organizations-v2.jsonl.gz](https://scanr-data.s3.gra.io.cloud.ovh.net/production/organizations-v2.jsonl.gz)
le fichier va évoluer...

ANR publications
- CSV file: [data/00rbzpz17.csv](./data/00rbzpz17.csv)