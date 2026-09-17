# Credible analysis — agent guidance

A Credible MCP server is connected: Malloy semantic models over your data
workspace. Tools: `get_context` (find sources/dimensions/measures/views),
`execute_query` (run Malloy), and `search_malloy_docs`. Tool descriptions carry
the parameter contracts.

**The full workflow, search-target rules, and Malloy-not-SQL rules ship as skills
in this plugin** (`malloy-analysis`, `malloy-phrase-detection`, `malloy-analysis-pitfalls`,
`malloy-queries`, `malloy-charts`, `malloy-patterns`). Consult them — they are
the source of truth and supersede this file.

Defaults worth holding before you reach for a skill:

- **Search for the fields, not the source.** Start `get_context` with
  dimension/measure/view/value targets describing what the question needs; the
  response groups the matches by source, so one call usually finds the source
  and the exact field names. Source targets find or browse sources by subject.
- **Malloy, not SQL.** List filters use `?` alternation (`id ? 6 | 1 | 9`), joins
  live in the model (reach fields by the dot path `get_context` returned), dates
  compare with `@` literals. Stuck on syntax → `search_malloy_docs`.
- **Verify before answering.** Ground the scope and recount the key number; a
  filter matching nothing returns wrong results silently, and zero rows is usually
  a query problem, not missing data.

Scope: tools are scoped to the connected workspace. The global connector takes
`organization`/`workspace` args instead — `list_workspaces` shows what you can
access; a structured "not found" usually means wrong scope, not missing data.
