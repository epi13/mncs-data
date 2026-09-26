# MNCS language pressure ledger

For each finding: the Data workload, current behavior, desired
behavior, reproducer, owning layer, workaround, and closure test. All
entries were hit building the canonical slice (`src/data`, profile
0.18) and are evidenced by compiler diagnostics, probes, or runtime
results from this campaign.

## P-DATA-001: no lambdas or closures for predicates (designed around)

- Workload: filter/map over rows with caller-supplied predicates.
- Current: predicates are descriptor data (`IntPred` six ways,
  one pinned column). Anything richer is inexpressible in-language.
- Desired: bounded closures or a documented descriptor idiom.
- Reproducer: `RowPred` in `src/data/ops.mncs` — the honest boundary;
  a `Fn`-style parameter does not exist to write.
- Owner: mncs-language (read-only this campaign).
- Workaround: explicit predicate enums; row mapping stays with
  callers outside Data.
- Closure: lambda/type-inference support with a filter-property test.
- Blocking: for expressive queries, yes; for the descriptor slice
  delivered here, no.

## P-DATA-002: generic inference fails across exact-to-view coercion

- Workload: `fnv.fold(state, exact_bytes, len)` where the parameter
  is `[byte; up_to N]`.
- Current: `MNE220` (cannot infer N); explicit `<...>` required even
  though the profile docs describe inference through exact bounds.
- Desired: inference through exact-to-view coercion, or documented
  limits.
- Reproducer: `src/data/image.mncs` before the turbofish fix;
  `/tmp/datasmoke/turbo.mncs` proves `fold<8>` works.
- Owner: mncs-language (read-only this campaign).
- Workaround: explicit `fold<8>` / `fold<32>` / `fold<256>`.
- Closure: inference success on the image module without turbofish.
- Blocking: no.

## P-DATA-003: sequence literals do not elaborate as call arguments

- Workload: `make_column([105, 100, ...], 2, ...)` with a concrete
  `[byte; 8]` parameter.
- Current: fails at the call site (`MNE117`/`MNE133` surfacing on the
  poisoned binding); typed lets work.
- Desired: literal elaboration against concrete sequence parameters,
  or a diagnostic at the literal.
- Reproducer: `/tmp/datasmoke/sortprobe.mncs` (first revision).
- Owner: mncs-language (read-only this campaign).
- Workaround: `c8`/`t32` chunk helpers plus typed lets everywhere.
- Closure: the probe compiling with inline literals.
- Blocking: no.

## P-DATA-004: float cells deferred (non-goal with a reason)

- Workload: scientific columns needing f64 with ordering semantics.
- Current: no Float cell; CSV interchange and the messy-data core
  (ids, counts, categories, flags) are fully served without one.
- Desired: a Numeric-coherent float design (ordering, NaN, tolerance
  discipline owned beside numerics, not improvised in Data).
- Reproducer: scope decision, not a diagnostic; see README non-goals.
- Owner: mncs-numeric + language (both read-only this campaign).
- Workaround: none in-slice; integers carry the interchange story.
- Closure: a float-cell RFC with Numeric-owned comparison semantics.
- Blocking: for float-heavy science workloads, yes; for the
  interchange/validation slice, no.

## P-DATA-005: no string literals (standing friction, paid again)

- Workload: column names, text spellings, CSV words.
- Current: `"` rejected at lexing (`MNL002`, evidenced last campaign);
  all spellings are numeric bytes via `c8`/`t32` chunk helpers.
- Desired: byte-string literals projecting to bounded arrays.
- Owner: mncs-language (read-only this campaign).
- Workaround: chunk helpers with comments.
- Closure: literal support with a schema-spelling test.
- Blocking: no.

## P-DATA-006: no enum equality (standing friction, projected around)

- Workload: comparing cell kinds, failure kinds, line statuses.
- Current: `==` on enums rejected (`MNE121`); decisions go through
  `match`.
- Desired: derived equality for field-less enums or documented idiom.
- Owner: mncs-language (read-only this campaign).
- Workaround: `match` internally; numeric projections
  (`kind_code`, `fail_code`, `line_code`, `pred_code`) as the stable
  host/test vocabulary.
- Closure: decision either way; projections stay useful.
- Blocking: no.

## P-DATA-007: views refuse span copy and functional update (designed around)

- Workload: buffer building from argv/line views.
- Current: `copy_span` needs an exact destination (`MNE265`);
  `replace` an exact base (`MNE198`).
- Desired: documented read-only-view rule or view-accepting forms.
- Owner: mncs-language (read-only this campaign).
- Workaround: exact buffers inside (`LineBuf`, `[byte; 32/64]`),
  views only at ingestion and transport.
- Closure: documentation or primitives.
- Blocking: no.

## P-DATA-008: reserved words and single-level projection (standing friction)

- Workload: ordinary bindings (`over`, `match`) and nested reads.
- Current: `over`/`match` rejected as bindings; `a.b.c` does not
  parse — intermediate lets required.
- Owner: mncs-language (read-only this campaign).
- Workaround: renames (`toolong`, `is_match`) and level-by-level lets.
- Closure: reserved-word documentation; nested projection or idiom.
- Blocking: no.

## External gaps observed (not language pressures)

- Doctor reports `DOC102: unknown profile 0.18` for all 9 `.mncs`
  files here, while the toolchain compiles and runs profile 0.18
  (same as sibling campaigns). Doctor's profile registry is stale;
  recorded here, fix belongs to mncs-doctor (read-only).
- Debug `import-test` cannot resolve out-of-tree suite sources
  (derives `target/tests/native/...` under the data root). Noted for
  mncs-debug (read-only); native results stand on their own.
- Store round-trip stays host-side: tables are persistable by value
  and `validate_table` is the restore gate, but no in-campaign run
  writes to Store (read-only). A Store-owned Data-image round-trip
  check is the natural follow-up, owned by Store.
