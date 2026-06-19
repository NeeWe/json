# nlohmann/json (JSON for Modern C++)

Single-header C++11 JSON library by Niels Lohmann. Header-only; no build required for consumers.

## Build & test

```sh
cmake -B build -DJSON_BuildTests=ON
cmake --build build
ctest --test-dir build
```

Tests use [Doctest](https://github.com/doctest/doctest) (bundled in `tests/thirdparty/doctest`).

### Common CMake options

| Flag | Default | Purpose |
|------|---------|---------|
| `JSON_BuildTests=ON` | OFF (OFF as subproject) | Enable test suite |
| `JSON_FastTests=ON` | OFF | Skip expensive tests (unicode4, etc.) |
| `JSON_MultipleHeaders=ON` | ON | Use multi-header (`include/`) vs amalgamated (`single_include/`) |
| `JSON_ImplicitConversions=OFF` | ON | Disable implicit type conversions |
| `JSON_Diagnostics=ON` | OFF | Extended error messages |
| `JSON_GlobalUDLs=OFF` | ON | Don't place `_json` literal in global namespace |
| `JSON_TestStandards=11` | (all) | Test only a single C++ standard |
| `JSON_ExportCompileCommands=OFF` | ON (as main project) | Generate `compile_commands.json` for clangd |
| `NLOHMANN_JSON_BUILD_MODULES=ON` | OFF | C++20 modules support (CMake ≥3.28) |

### Run a focused test

Test naming: `test-<name>_cpp<standard>` (e.g., `test-regression1_cpp11`).

```sh
ctest --test-dir build -R test-regression1_cpp11 --output-on-failure
```

Test file → name mapping: `tests/src/unit-<name>.cpp` → `test-<name>`. See `cmake/test.cmake:215`.

### Skip expensive tests

```sh
cmake -B build-fast -DJSON_BuildTests=ON -DJSON_FastTests=ON
cmake --build build-fast
ctest --test-dir build-fast
```

`JSON_FastTests=ON` skips the slow `test-unicode4` (and sets `--no-skip` is omitted so doctest `TEST_CASE` with `*` is skipped).

### Test a single C++ standard

```sh
cmake -B build-cpp17 -DJSON_BuildTests=ON -DJSON_FastTests=ON -DJSON_TestStandards=17
cmake --build build-cpp17
ctest --test-dir build-cpp17
```

### Valgrind

```sh
cmake -B build-vg -DJSON_BuildTests=ON -DJSON_Valgrind=ON
cmake --build build-vg
ctest --test-dir build-vg -L valgrind
```

## Source layout

| Path | Purpose |
|------|---------|
| `include/nlohmann/` | Multi-header source (default at build time) |
| `single_include/nlohmann/json.hpp` | Amalgamated single header (what users consume) |
| `tests/src/unit-*.cpp` | Test sources, one per feature area |
| `tools/amalgamate/` | Python script for generating single header |
| `cmake/ci.cmake` | All CI test targets (compiler variants, sanitizers, coverage, etc.) |

Regenerate single header: `make amalgamate` (requires astyle venv).

## Formatting

Two options:
- **astyle**: `make pretty` (uses `tools/astyle/venv`)
- **clang-format**: `make pretty_format`

CI enforces amalgamation and formatting via `ci_test_amalgamation`.

## Test data

Test JSON files are downloaded automatically from `nlohmann/json_test_data` (tag v3.1.0) during CMake configure via `ExternalProject_Add`. Set `-DJSON_TestDataDirectory=/path` to use local copy.

## CI

- **Cirrus CI** (`.cirrus.yml`) — ARM Linux, fast tests
- **AppVeyor** (`.github/external_ci/appveyor.yml`) — Windows
- **GitHub Actions** — Ubuntu, macOS, Windows workflows
- All CI targets defined in `cmake/ci.cmake` (e.g., `ci_test_gcc`, `ci_test_clang`, `ci_test_coverage`, `ci_test_amalgamation`)

## Fuzz testing

AFL-based fuzzers for JSON, BSON, CBOR, MessagePack, UBJSON, BJData parsers:

```sh
make fuzz_testing      # builds parse_afl_fuzzer
```

Fuzzer sources in `tests/src/fuzzer-parse_*.cpp`.

## Notable quirks

- `JSON_ImplicitConversions` and `JSON_Diagnostics` are CMake options that set `#define`s on the interface target — consumers inherit them.
- C++ modules (`import nlohmann.json`) require CMake ≥3.28 and `-DNLOHMANN_JSON_BUILD_MODULES=ON`.
- The `json_fwd.hpp` forward declaration header is also amalgamated.
- MSVC builds need `/bigobj` and `/STACK:4000000` for some binary-format tests (set automatically by CMake).
- Apple CLT 26+ may be missing C++ standard headers in the default include path. CMakeLists.txt auto-detects this and falls back to the SDK's `c++/v1`. If you see "ctime" or "iosfwd" not found, pass `-DCMAKE_OSX_SYSROOT=$(xcrun --show-sdk-path)`.
- `.vscode/settings.json` configures clangd to use `build/compile_commands.json` for precise navigation. Enable with `-DJSON_ExportCompileCommands=ON` (default when building as main project).
