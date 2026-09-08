# Agent and contributor contract

- Prefer `mncs-language` for implementation.
- Keep common data transformations concise while retaining explicit type/error semantics.
- Do not erase malformed, missing or ambiguous data merely to simplify APIs.
- Distinguish schema inference from schema validation.
- Preserve streaming/lazy behavior where declared; do not accidentally materialize unbounded inputs.
- Record language/compiler/runtime friction in `docs/LANGUAGE_PRESSURES.md`.
- Test realistic dirty data as well as clean fixtures.
