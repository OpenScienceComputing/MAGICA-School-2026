@AGENTS.md

## Claude Code

For editing `.ipynb` files, use the Jupyter MCP — it runs against a live kernel so you can execute cells and verify real outputs, and it writes through nbformat so git diffs stay clean. Do not use the built-in `NotebookEdit` tool: it cannot execute code and collapses cell source into a single JSON string, which mangles notebook diffs. For a non-interactive "does it run?" check where you don't need the MCP, use the `nbconvert --execute` command in [AGENTS.md](AGENTS.md).

Task-specific lessons live in skills under `.claude/skills/` and load automatically when relevant. For example, `building-shyfem-stac` covers building STAC Items/Catalogs for SHYFEM UGRID Icechunk stores (Icechunk auth, xstac quirks, rustac, storage extension, etc.).
