# System Overview

This document outlines the components required to turn the `CRISPR-All` prototype into a full application.

## 1. System Architecture

| Layer | Core responsibility | Key tech choices |
|-------|--------------------|-----------------|
| **Client (browser)** | Interactive UI for module selection, natural-language entry and multi-cassette design | React + TypeScript |
| **API gateway** | Stitches together gene lookup, LLM parsing, sequence rendering and file export | FastAPI (Python) with async calls and JWT auth |
| **Micro-services** | Gene service fetches sequences. Assembler builds cassettes. Validation checks ORFs, frame, GC and cloning constraints | Dockerised services: Python for gene, BioPython for assembler, Rust for validation |
| **Data** | Local mirror of human gene library plus standardised module sets and backbones | PostgreSQL (metadata) + S3 bucket (FASTA/GenBank blobs) |

## 2. Data Sources and Pre-loading

- Mirror canonical gene references once per release using Ensembl REST (`rest.ensembl.org`)
- Fallback to NCBI EFetch for any missing loci
- Store two tables in PostgreSQL:
  - `genes(hgnc_symbol, ensembl_id, length, cds_sequence, synonyms[])`
  - `modules(type, name, sequence, default_connector, annotations jsonb)`
- Version each release so users can reproduce designs

## 3. Backend Services

### Gene service
- Endpoint: `/lookup?symbol=SYMBOL` returns metadata and CDS
- Caches queries in Redis for speed

### Connector engine
- Provides default connectors such as P2A peptides or U6 promoters
- Allows per-junction overrides

### Assembler
- Accepts ordered lists of elements and connectors
- Inserts intron and adaptor stubs using a universal template
- Generates FASTA and annotated GenBank output

### Multi-construct manager
- Handles arrays of cassettes
- Validates barcodes, plasmid length and cloning enzymes
- Exports a `.tar.gz` bundle of FASTA files plus an index CSV

## 4. Front-end Features

- Natural language mode that sends a prompt to an LLM and receives a JSON plan
- Autocomplete search over the human gene list (`/genes/suggest` endpoint)
- Drag-and-drop cassette builder with peptide selector and trash bin
- Tabs for multiple cassettes and real-time sequence preview
- Export buttons for FASTA, GenBank and pooled archives

## 5. LLM Integration

- Prompt templates tuned on internal examples
- Post-process responses with the Validation service before presenting results
- Rate limiting and caching to handle spikes

## 6. Constants

- Universal adaptors, splice sites and barcode templates live in a Python module
- Backbone signature: pLL3.7-derived 7.2 kb plasmid with BsaI site

## 7. Generating Annotated GenBank

Example in Python using BioPython:

```python
from Bio.SeqRecord import SeqRecord
from Bio.SeqFeature import SeqFeature, FeatureLocation
# ... construct record and write
```

## 8. Multi-cassette Pooled Screens

- Enforce max cassette count per pool
- Assign unique 10–12 bp barcodes and create 96‑well plate maps
- Provide “balanced” mode to normalise cassette lengths
