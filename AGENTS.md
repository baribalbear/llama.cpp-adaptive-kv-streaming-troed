# jcodemunch-mcp — Workflow Guide

Use these tools instead of Grep/Read/search for any indexed repository.

## Step-by-step

1. **list_repos** — check if the project is already indexed.
   - If not found, run **index_folder** (local) or **index_repo** (GitHub URL).

2. **search_symbols** — find functions, classes, methods by name or description.
   - Use `detail_level: "full"` to get source inline, or follow up with **get_symbol_source**.

3. **get_context_bundle** — get symbol source + its imports in one call.

4. **search_text** — fall back to full-text / regex search for string literals or comments.

5. **get_file_outline** — list all symbols in a file without reading the whole thing.

