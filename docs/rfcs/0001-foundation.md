# RFC 0001: Data pipeline foundation

Status: Implemented (2026-09-26: typed cells, schemas, validation, table ops, CSV interchange, digests, and native suites landed)

## Principles

- Real-world bad data is represented, not silently discarded.
- Schemas can be explicit; inference is a separate fallible operation.
- Transformation APIs should be fluent and concise without becoming dynamically untyped.
- Lazy/streaming execution preserves observable semantics.
- Columnar and row execution may differ physically while results remain equivalent.
- Parse/validation errors retain location and useful context.

## Pressure objectives

Records and structural typing, heterogeneous values, option/result types, lambdas, type inference, iterators/streams, lazy evaluation, generics, schema metadata, Unicode, date/time types, memory-efficient buffers and diagnostics for transformation/type errors.
