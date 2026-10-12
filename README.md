# templates

Opinionated, minimal project templates, one repository per language, collected here as git submodules.

| Template | Stack | Coverage | Mutation testing |
|---|---|---|---|
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
| Svelte / TypeScript | ESLint `complexity`: 10 | SonarJS `cognitive-complexity`: 15 (script functions, not template markup) |
| Ada / Zig | Not configured in the current toolchains | Not configured in the current toolchains |

Ada's GNATcheck offers a cyclomatic-complexity rule, but is not supplied by the current
Alire dependencies/index. Zig currently uses compiler checks and `zig fmt`.

```sh
git clone --recurse-submodules https://github.com/Napolitain/templates
git submodule update --remote    # move every submodule to its latest main
```

## Profile-guided optimization

PGO is opt-in. Follow the commands in each template's README after replacing the
starter with your application and choosing a training workload.

| Template | Instrumentation PGO | Sampling PGO / AutoFDO | Propeller / relinking |
|---|---|---|---|
| [C++](template-cpp/README.md#profile-guided-optimization) | Clang generate/use presets | Clang supports it; external Linux profiling/conversion tools required | External experiment; no supplied workflow |
| [Rust](template-rust/README.md#profile-guided-optimization) | Stable rustc generate/merge/use commands | No supported stable workflow here | No supplied workflow |
| [Go](template-go/README.md#profile-guided-optimization) | Not used by the Go compiler's PGO workflow | Go CPU pprof profiles, explicit `-pgo` | Not supported by this template's linker workflow |
| [Ada](template-ada/README.md#profile-guided-optimization) | GCC-based GNAT generate/use commands | GCC AutoFDO requires a separately configured toolchain/converter | No supplied workflow |
| [Zig](template-zig/README.md#profile-guided-optimization) | No supported Zig-source workflow | No supported workflow | No supplied workflow |
| [Python](template-python/README.md#profile-guided-optimization) | Interpreter build option, not a Python-package build mode | No application build workflow | Interpreter-level tooling only |
| [Svelte / TypeScript](template-svelte/README.md#profile-guided-optimization) | No Vite application build mode | No application profile-use mode | Not applicable to the emitted JS/CSS |

The greeting examples only verify that profile collection and consumption work.
Train on realistic input sizes, hot paths, error rates and concurrency, in their
expected proportions. Unit tests and coverage profiles are not representative
performance profiles. Let instrumented processes exit normally to flush counters;
long-running services need their profiler's explicit flush/stop mechanism.

Keep the source revision, compiler, target, CPU flags, dependency lock and workload
fixed between generation and use. Start fresh when these change; never merge
profiles from incompatible binaries. Record those inputs alongside retained
profiles. Scratch profiles live under ignored build directories and are not
required by normal builds, hooks or CI.

Check correctness again after rebuilding. Benchmark the baseline and optimized
release binaries on the same machine with warmups and repeated runs, using a
separate representative evaluation workload. Compare elapsed time or throughput,
tail latency where relevant, memory and binary size; retain PGO only when it helps.
Native CPU defaults constrain which machines can run these binaries. No speedup
is claimed for the greeting programs.

[Propeller](https://github.com/google/llvm-propeller) additionally needs compatible
Clang/LLD, a binary with basic-block address maps, hardware branch sampling and a
profile converter producing compiler and linker layout profiles. An LLVM backend
or LTO alone does not establish support. These templates do not ship an unverified
relinking preset; assess that separately on the intended OS, CPU and toolchain.
