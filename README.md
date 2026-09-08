# mncs-data

High-level machine-native data manipulation for MNCS.

`mncs-data` pressures `mncs-language` with the messy application workloads that scientific and systems code often avoid: partial schemas, missing values, heterogeneous records, Unicode, dates, malformed input, evolving data, lazy pipelines and large streaming datasets.

## Initial scope

- records, schemas, tables and columns
- CSV, JSON and other structured interchange boundaries
- parsing, validation and schema inference
- select/filter/map/group/join/aggregate pipelines
- missing/null/error values
- lazy and streaming execution
- row/column-oriented representations
- provenance and transformation evidence

## Repository layout

- `docs/ARCHITECTURE.md`
- `docs/rfcs/0001-foundation.md`
- `docs/LANGUAGE_PRESSURES.md`
- `AGENTS.md`
