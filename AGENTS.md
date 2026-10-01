# jcodemunch-mcp — Workflow Guide

Use these tools instead of Grep/Read/search for any indexed repository.

## Step-by-step

1. **list_repos** - check if the project is already indexed.
   - If not found, run **index_folder** (local) or **index_repo** (GitHub URL).

2. **search_symbols** - find functions, classes, methods by name or description.
   - Use `detail_level: "full"` to get source inline, or follow up with **get_symbol_source**.

3. **get_context_bundle** - get symbol source + its imports in one call.

4. **search_text** - fall back to full-text / regex search for string literals or comments.

5. **get_file_outline** - list all symbols in a file without reading the whole thing.

## Build and test

Build and test commands run through the host allowlist (`umwelt_action`), not the
sandbox shell. The sandbox has no `nvcc`, so a direct `cmake` fails at
configuration with "Failed to find nvcc". The host actions export
`PATH=/usr/local/cuda-13/bin` first.

| Goal | `umwelt_action name` |
| --- | --- |
| Incremental CUDA Release build (`build/`) | `llama-build` |
| Same, detached; log `build-bg.log` | `llama-build-bg` |
| CPU test build (`build-tests/`) | `llama-build-tests` |
| CUDA test build (`build-tests-cuda/`) | `llama-build-tests-cuda` |
| Run the fork's 9 kv-stream tests | `llama-test` |
| Restart the router with the new build | `llama-restart` |
| GPU memory used/total | `llama-gpu` |

Detached builds return at once. Wait for the `BUILD_EXIT=` / `TESTS_BUILD_EXIT=`
marker with `wait-build.sh <base-marker-count> <log>`:

```
sh .superpowers/sdd/2026-10-01-streamed-sizing-contract/wait-build.sh 0 build-bg.log
```

Read the marker count BEFORE launching. A count read after launch never changes,
and the poll hangs.

Direct commands, when a host action does not fit:

```
cmake --build build --config Release -j 12
cmake -S . -B build-tests -DLLAMA_BUILD_TESTS=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build-tests --config Release -j 12
ctest --test-dir build-tests --output-on-failure
```

`build/` and `build-tests-cuda/` have `GGML_CUDA=ON`; `build-tests/` is CPU only.
`build-tests/` holds 106 tests; the fork's subset selects 9 kv-stream and
arg-parser tests.

`build-tests-cuda/` can fail at the CMake generate step with
"CUDA_ARCHITECTURES is empty" when its cache is stale. Delete the directory and
reconfigure to recover. Use `build-tests/` for the routine suite.

## Measurement

`src/llama-context.cpp` prints one `memory_phase:` line per phase at WARN level:

```
memory_phase: phase=decode compute=... kv_pool=... writer=... attention=... resident_pages=...
```

Start the server with `--log-verbosity 4`, run a request, then read the lines for
the phase of interest. Do not trust notes; take figures from these lines.

