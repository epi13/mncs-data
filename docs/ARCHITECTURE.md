# Architecture

## Layers

1. **Values/schema** — scalar values, records, nullable/error cells and schema representation.
2. **I/O** — CSV/JSON and future structured formats with streaming parsers/emitters.
3. **Collections** — row, column and chunk abstractions.
4. **Transforms** — projection, filtering, mapping, grouping, joining, aggregation and sorting.
5. **Planning/execution** — eager/lazy pipelines, pushdown/fusion opportunities and bounded memory.
6. **Validation/evidence** — schema checks, rejected-row information, provenance and deterministic tests.

## First milestones

1. Typed records plus CSV/JSON streams.
2. Missing/error-value semantics.
3. Core transformation pipeline.
4. Group/join/aggregate operations.
5. Lazy/streaming planner and large-data pressure cases.
