# mncs-data

The smallest coherent typed data layer MNCS actually needs: explicit
schemas, honest missing/invalid cell semantics, schema validation with
structured failures, small composable table transforms, deterministic
CSV interchange, and content digests for the Store/Lineage boundary.

Data owns **typed interchange values and their transformation
semantics**. It does not own persistence (Store), project contracts
(Commons), provenance graphs (Lineage), or language-level collections
(stdlib). Tables persist by value; digests and transformation notes are
the integration surface — never a second database.

## What Data owns

- `Cell`: Int / Text / Bool values plus **Missing** (no value present)
  and **Invalid(code)** (malformed, with reason) — four states that are
  never collapsed (`src/data/cell.mncs`)
- explicit 4-column schemas with required flags (`src/data/schema.mncs`)
- generic schema validation with structured failures carrying
  (row, column, kind, detail) (`src/data/validate.mncs`)
- bounded tables: 4 columns, at most 16 rows, explicit overflow
  (`src/data/table.mncs`)
- relational primitives: predicate filter, stable sort with
  Missing-last, narrowing projection, sums/extents/counts with open
  Missing-skipping (`src/data/ops.mncs`)
- deterministic CSV emission and bounded, never-trapping CSV parsing
  (`src/data/csv.mncs`)
- FNV-1a content digests and transformation notes
  (`src/data/image.mncs`)

## What Data does not own

| Concern | Owner | Data posture |
|---|---|---|
| persistence, durable records | Store | tables persist by value; `validate_table` is the load-time gate |
| project contracts, identities | Commons | no duplicate schema registry |
| derivation graphs | Lineage | emits digests + notes; owns no graph |
| records, arrays, iteration, sorting u64s | language/stdlib | consumes; never re-implements |
| JSON | stdlib (`json_emit`/`json_cursor`) | not duplicated |
| Float semantics | deferred (see non-goals) | no Float cell in this slice |

## Data model

A row is 4 heterogeneous cells against a schema; a table is a schema
plus up to 16 rows. Values are self-contained (fixed scalar arrays, no
references). Missing ≠ Invalid ≠ zero ≠ empty text ≠ false. Schemas
are explicit data — inference is out of scope by contract. One schema
describes one table shape; canonical schemas upgrade normally with all
consumers in-repo, so there is no version ladder.

## Validation

`validate_row` is generic over the column width. Failures: 1 =
MissingRequired, 2 = TypeMismatch (detail: expected kind), 3 =
InvalidCell (detail: cell code). At most 8 failures record per row;
further failures still fail. `validate_table` reports the first bad
row with (row, column, code) — the Debug context for a load failure.

## Transformations

Predicates are explicit descriptors (`IntPred`: Eq/Ne/Gt/Lt/Ge/Le on
one column) — the language has no lambdas, so filters say exactly
this much, openly. Only present Int cells match; Missing/Invalid never
satisfy a predicate. Sort is a stable selection permutation with
unordered keys last in both directions. Aggregates skip Missing openly
and count Invalid separately. Projection narrows 4 → 3/2/1 columns.

## CSV boundary (canonical encoding)

`Int` decimal (no `+`, no spaces) · `Text` raw bytes (no quoting;
`"` passes through) · `Bool` true/false · `Missing` empty field ·
`Invalid` `!` + code. Header carries column names; callers skip it on
input. Lines are ≤ 256 bytes; callers flag truncation instead of
losing it. Short rows pad Missing; long rows report Ragged and keep
their first 4 cells; empty lines report Empty. Text past 32 bytes is
Invalid(3), never silently cut. Text containing commas re-parses as
Ragged — quoting is a documented non-goal, silent corruption is not
on the table.

## Determinism

Emission is byte-exact (pinned by tests, including i64 MIN/MAX).
Digests fold exact spellings, so equal values digest equally and
column/row order matters. `sorted_emit_order_free` proves input order
independence end to end.

## Boundedness

4 columns · 16 rows · 32-byte texts · 256-byte lines · 8 recorded
failures · 19-digit integers. Every limit reports (overflow, Ragged,
Truncated, Invalid) instead of trapping or dropping.

## Verification

```
python3 scripts/run_tests.py                 # both suites via `mncs test`
python3 scripts/run_tests.py --suites datacore
python3 scripts/run_tests.py --suites dataio
```

49 native assertions: construction, validation (exact failure codes),
missing/optional semantics, filter/sort/project/aggregate, overflow,
byte-exact emission, 8-case generated round-trip properties
(`mncs.test.generative`), malformed inputs (bad int/bool, ragged,
truncated, overlong, CR, spaces, `+` signs), digest determinism, and
the Store-shape load path (parse → push → validate_table).

## Non-goals (explicit)

Float cells (needs Numeric-coherent ordering/NaN design) · schema
inference · joins/group-by (predicate/filter/sort cover the slice;
real demand becomes pressure) · quoting/multiline CSV · streaming
past the 16-row chunk contract · schema version ladders.

## Pressures

`docs/LANGUAGE_PRESSURES.md` records eight evidenced findings:
lambdas for predicates, explicit generic args at view boundaries,
inline literals as call args, float-cell deferral, and the standing
frictions (no strings, no enum equality, reserved words, view-mutation
refusals). Doctor reports `DOC102: unknown profile 0.18` for every
`.mncs` file here; the toolchain runs profile 0.18 (as in sibling
campaigns), so this is Doctor's stale registry, not a source defect.
