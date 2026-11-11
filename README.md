# templates

See [AGENTS.md](AGENTS.md) for the working, verification, performance and commit policies.

Opinionated, minimal project templates, one repository per language, collected here as git submodules.

| Template | Stack | Coverage | Mutation testing |
|---|---|---|---|
| [template-angular](https://github.com/lab34/template-angular) | Angular 22, TypeScript (zoneless), Tailwind CSS, pnpm, ESLint, Prettier, Playwright | Vitest V8 | |
| [template-ada](https://github.com/Napolitain/template-ada) | Ada 2022 + SPARK (gnatprove), Alire, GNAT, AUnit, GNATformat | GNATcoverage | |
| [template-cpp](https://github.com/Napolitain/template-cpp) | C++26, clang, CMake presets, FetchContent, GoogleTest, clang-tidy, cppcheck | llvm-cov | |
| [template-go](https://github.com/Napolitain/template-go) | Go, gofumpt, go vet, golangci-lint, deadcode | go test -cover | gremlins |
| [template-python](https://github.com/Napolitain/template-python) | Python, uv, ruff, ty, pytest | pytest-cov | mutmut |
| [template-rust](https://github.com/Napolitain/template-rust) | Rust 2024, clippy pedantic, rustfmt | cargo-llvm-cov | |
| [template-svelte](https://github.com/Napolitain/template-svelte) | Svelte 5, TypeScript, Vite, Tailwind CSS, pnpm, ESLint, Prettier, Playwright | Vitest V8 | |
| [template-zig](https://github.com/Napolitain/template-zig) | Zig, zig fmt, zig build test | kcov | |

Shared conventions: hooks via [prek](https://prek.j178.dev) with every available autofix enabled, 4-space indentation (`.editorconfig`), native CPU builds with LTO and stripping for release where applicable, CI on GitHub Actions (`ubuntu-26.04`), latest compatible tool versions.

To start a project, use a template directly (`gh repo create my-app --template Napolitain/template-go`), not this repository.

Each template includes an optional Nix development shell using
`github:NixOS/nixpkgs/nixpkgs-unstable`. Installing tools directly on NixOS remains
supported; Nix is not required by the hooks or CI. From a template directory:

```sh
nix develop --command "$SHELL"       # keep your shell, aliases and prompt
nix flake update nixpkgs             # refresh the lock to latest unstable
```

Re-enter the shell after updating. Each template owns its flake and lock file,
so it works independently when used to create a project. See its README for
platform support and toolchain details.

Complexity limits run through each template's existing lint hooks and CI. Where supported,
the maximum is 10 for cyclomatic complexity and 15 for cognitive complexity; violations
require refactoring. Algorithms differ between tools, so scores are not directly comparable.

| Template | Cyclomatic | Cognitive |
|---|---|---|
| Go | `cyclop`: 10 | `gocognit`: 15 |
| Python | Ruff `C901`: 10 | Not provided by Ruff |
| Rust | No separate Clippy rule | Clippy `cognitive_complexity`: 15 (Clippy-specific heuristic) |
| C++ | No configured gate | clang-tidy `readability-function-cognitive-complexity`: 15 |
| Svelte / TypeScript / Angular | ESLint `complexity`: 10 | SonarJS `cognitive-complexity`: 15 (script functions, not template markup) |
| Ada / Zig | Not configured in the current toolchains | Not configured in the current toolchains |

Ada's GNATcheck offers a cyclomatic-complexity rule, but is not supplied by the current
Alire dependencies/index. Zig currently uses compiler checks and `zig fmt`.

```sh
git clone --recurse-submodules https://github.com/Napolitain/templates
git submodule update --remote    # move every submodule to its latest main
```

## Profile-guided optimization

Profile optimization is opt-in; keep an ordinary release baseline and select a
mode supported by your template. Each linked README contains runnable commands.

| Template | Easy path | Sampling / code layout |
|---|---|---|
| [C++](template-cpp/README.md#profile-guided-optimization) | `tools/perf-build baseline` / `pgo -- ./train.sh` | `autofdo`, `propeller`, `propeller-pgo`, `propeller-autofdo`; Linux perf + LLVM |
| [Rust](template-rust/README.md#profile-guided-optimization) | `tools/perf-build baseline` / `pgo -- ./train.sh` | Instrumentation PGO; no stable AutoFDO/Propeller template mode |
| [Ada](template-ada/README.md#profile-guided-optimization) | `tools/perf-build baseline` / `pgo -- ./train.sh` | GNAT instrumentation; no supplied AutoFDO/Propeller mode |
| [Go](template-go/README.md#profile-guided-optimization) | Explicit `go build -pgo=off` / `-pgo=cpu.pprof` | Native sampled CPU pprof; no LLVM/Propeller mode |
| [Zig](template-zig/README.md#profile-guided-optimization) | Release full LTO | No supported Zig-source PGO workflow |
| [Python](template-python/README.md#profile-guided-optimization), [Svelte](template-svelte/README.md#profile-guided-optimization), [Angular](template-angular/README.md#profile-guided-optimization) | Normal app builds | No application PGO/Propeller build mode |

C++, Rust and Zig release builds retain full/fat LTO; Ada retains GCC LTO
(`-flto=auto` selects parallelism, not ThinLTO). LTO does not itself provide PGO.
The helpers require an executable workload driver, called with the training binary
as its last argument. In `train.sh`, run `"$1"` with your application's realistic
inputs. Instrumented processes must exit normally to flush profiles. All profiles
and optimized outputs stay in ignored build directories; normal builds need none.

### Compare before adopting

1. Train on typical inputs and concurrency; evaluate on **separate** representative
   inputs, including important cold/error paths. The greetings only test the pipeline.
2. Keep source, compiler, dependencies, LTO and CPU flags fixed. Check correctness
   of the final optimized binary. Retrain after those inputs or the workload change.
3. Compare baseline/optimized time or throughput with warmups and repeated runs;
   reverse their order too. Check tail latency, peak memory, binary size and build cost.
4. Retain the baseline if a critical workload regresses or gains fall within noise.
   No benchmark proves there are no disadvantages for every possible workload.

From `template-cpp/`, with an executable `evaluate.sh` accepting the binary path:

```sh
hyperfine --warmup 3 --runs 20 --export-json build/pgo/comparison.json \
    './evaluate.sh ./build/pgo-baseline/template_example' \
    './evaluate.sh ./build/autofdo-use/template_example'
perf stat -r 5 -e cycles,instructions,branches,branch-misses -- \
    ./evaluate.sh ./build/autofdo-use/template_example
```

Run the counters on both variants. `perf stat` measures; AutoFDO/Propeller use
`perf record` and converted profiles to change the build. Percent time saved is
`100 * (baseline - optimized) / baseline`.

**Ballpark:** gains can range from none (or regressions) to low double digits.
[Go documents 2–14%](https://go.dev/doc/pgo) for its representative benchmark set
as of Go 1.22; [Rust reports 15–20% for rust-analyzer](https://blog.rust-lang.org/2025/09/10/rust-compiler-performance-survey-2025-results/).
These are application examples, not predictions; PGO, AutoFDO and Propeller gains
are not additive. I/O-bound work may see little wall-clock improvement.
