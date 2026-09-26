# Architecture

## Layers (realized)

1. **Values** (`src/data/cell.mncs`) — `Cell` (Int/Text/Bool/Missing/
   Invalid) with checked constructors, kind predicates, exact equality,
   Int comparison ranks, and total payload projections (`as_int`,
   `as_bool`). No Float cell (explicit non-goal).
2. **Schemas** (`src/data/schema.mncs`) — `ColumnSpec` (name ≤ 8 bytes,
   kind, required) and `Schema4`; name lookup with deterministic
   last-wins duplicates. Explicit data, never inferred here.
3. **Validation** (`src/data/validate.mncs`) — generic
   `validate_row<C>` over `[ColumnSpec; C]`/`[Cell; C]` plus the
   `validate_row4` convenience entry. Bounded failure window (8).
4. **Tables** (`src/data/table.mncs`) — `Table` (schema + ≤ 16 rows),
   `push_row` with explicit overflow, `column_vector`, and
   `validate_table` as the Store load-time gate. Self-contained
   values: persistable by construction.
5. **Transforms** (`src/data/ops.mncs`) — descriptor predicates
   (`IntPred`/`RowPred`), `filter_table`, `sort_table` via an explicit
   selection permutation (`sort_order` + `permute`, stable,
   Missing-last, direction-stable ties), concrete narrowing
   projections (`project1/2/3`), `sum_int`/`extent_int` with open
   Missing-skipping, generic `count_states<N>`.
6. **Interchange** (`src/data/csv.mncs`) — `LineBuf` (256-byte exact
   window + length + poisoning `ok`), byte-exact emission
   (`emit_row`, `emit_header`, `put_i64` total at MIN), and bounded
   schema-directed parsing (`parse_typed` → `ParseRow` with
   Ok/Ragged/Truncated/Empty status). Never traps, never drops rows
   silently.
7. **Boundary image** (`src/data/image.mncs`) — FNV-1a digests over
   exact spellings (`schema_digest`, `row_digest`, `table_digest`,
   `bytes_digest256`) and `TransformNote` op/input/output triples for
   Lineage filing. No graph semantics here.
8. **Testing** (`tests/native/`, `scripts/run_tests.py`) — `datacore`
   (29 assertions) and `dataio` (20 assertions) through canonical
   `mncs test`, including `mncs.test.generative` round-trip
   properties; evidence envelopes carry run identity.

## Data flow

```
CSV line --parse_typed--> cells --validate_row--> ok/failures
cells --push_row--> Table --validate_table--> load report
Table --filter/sort/project--> Table (+ TransformNote via image)
Table --column_vector--> cells --sum/extent/count--> scalars
row --emit_row--> bytes --parse_typed--> row (round trip)
```

## Key decisions

- Fixed 4×16 canonical window with generic algorithms inside
  (`validate_row<C>`, `count_states<N>`): concrete types where the
  language needs them, generics where it supports them.
- Predicate descriptors instead of closures: the honest shape without
  lambdas, and the sharpest pressure for them.
- Parity of failure: every bound reports (overflow, Ragged,
  Truncated, Invalid, MissingRequired, TypeMismatch) — the language's
  totality made concrete for data.
- Digests and notes, not graphs: the Lineage integration that fits
  today's owners.
- CSV is owned as a boundary because Store explicitly declines it as
  a canonical model; JSON stays with stdlib and is not duplicated.
