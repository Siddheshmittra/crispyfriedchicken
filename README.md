# CRISPR-All Design Lab

This repository contains a prototype UI for building CRISPR cassette designs. The original implementation is a single HTML file (`CRISPRALL.html`).

## Proposed architecture

To expand the tool into a full web application, the following architecture is proposed:

1. **Client** – React + TypeScript front end that offers interactive module selection, natural language entry and a multi-cassette builder.
2. **API Gateway** – FastAPI service providing JWT-authenticated endpoints for gene lookup, large language model (LLM) parsing, sequence rendering and file export.
3. **Micro-services**
   - **Gene service** (Python + requests) fetches sequences from a local gene mirror or external APIs.
   - **Assembler service** (BioPython) constructs cassettes from ordered modules, inserting adaptors and generating FASTA/GenBank output.
   - **Validation service** (Rust) checks open reading frames, GC content and cloning constraints.
4. **Data layer** – PostgreSQL for relational metadata and an object store (e.g., S3) for FASTA/GenBank blobs. A local mirror of canonical gene references is preloaded for performance and reproducibility.

Additional details for each component are included in `docs/system_overview.md`.

## Usage

Open `CRISPRALL.html` in a browser to view the static prototype. The new architecture is not yet implemented in code.
