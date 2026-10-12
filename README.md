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

### Keeping builds with and without PGO

The recipes retain two release binaries; neither is the instrumented training
binary. Paths below are relative to the corresponding template directory:

| Template | Without PGO | With PGO |
|---|---|---|
| C++ | `cmake --preset pgo-baseline` then `cmake --build --preset pgo-baseline`; `build/pgo-baseline/template_example` | `pgo-use` configure/build presets; `build/pgo-use/template_example` |
| Rust | Baseline recipe without `-Cprofile-use`; `target/pgo-baseline/<host>/release/template` | Profile-use recipe; `target/pgo-use/<host>/release/template` |
| Go | `go build -pgo=off -o bin/pgo/baseline ./cmd/template-go` | `go build -pgo="$PWD/bin/pgo/cpu.pprof" -o bin/pgo/optimized ./cmd/template-go` |
| Ada | `alr build --release -- -f --subdirs=pgo-baseline`; `bin/pgo-baseline/template` | Profile-use recipe; `bin/pgo/template` |

Follow each README's collection/merge steps before building with PGO. Default
builds stay unchanged, and the baseline remains available as a fallback.

### Checking benefits and costs

No finite benchmark proves that PGO has no disadvantages for every possible
input or machine. It changes optimization choices, and improving a hot path can
come at the expense of another path or code size. Instrumentation adds overhead
to training runs; collecting profiles, rebuilding and refreshing them also cost
time and storage. Measure the final profile-use binary, not the instrumented one.

Use this comparison process for each application:

1. Build both variants from the same source and dependencies with identical
   release, LTO and CPU settings, changing only PGO. Record the compiler, revision,
   hardware, flags, profile identity and workload inputs with the results.
2. Train on typical work, then evaluate on separate inputs covering common paths,
   large inputs, error cases and expected concurrency. Check identical results and
   run correctness tests for both builds.
3. Measure repeated runs on a quiet machine, alternate which variant runs first,
   and separate cold-start tests from warmed-up steady-state tests. Save individual
   timings; investigate variation rather than assuming a small mean difference is
   real. For services, measure throughput and tail latency under the same load.
4. Compare each workload, peak memory, binary size, build time and training cost.
   A weighted average can summarize the expected traffic mix, but must not hide a
   regression on a critical workload.
5. Set tolerances before measuring. Select PGO only when a repeatable improvement
   exceeds measurement noise and important workloads remain within their limits.
   Keep the baseline if results are inconclusive; repeat after compiler, workload
   or significant source changes. This validates the tested cases, not all inputs.

For example, after building the C++ variants, run from `template-cpp/`:

```sh
hyperfine --warmup 5 --runs 30 --export-json build/pgo/comparison.json \
    './build/pgo-baseline/template_example' './build/pgo-use/template_example'
hyperfine --warmup 5 --runs 30 --export-json build/pgo/comparison-reversed.json \
    './build/pgo-use/template_example' './build/pgo-baseline/template_example'
```

Replace both greeting commands with the same meaningful application workload.
These commands collect timings; they do not automatically certify a performance
win. For elapsed time, report `100 * (baseline - pgo) / baseline` as percent time
saved. This differs from percent throughput gained; label the measured metric.

### Performance expectations

Published results give context, not a promise for a new project:

- [Go's PGO guide](https://go.dev/doc/pgo) reports about 2-14% performance
  improvement across its representative benchmark set as of Go 1.22.
- [The Rust project](https://blog.rust-lang.org/2025/09/10/rust-compiler-performance-survey-2025-results/)
  reports 15-20% performance wins for PGO builds of rust-analyzer. That is one
  substantial application, not a general guarantee for Rust programs.

These are examples of gains from a few percent to the low double digits. A new
application can see no measurable gain or a regression; there is no defensible
universal percentage for C++, Rust or Ada. Work dominated by I/O or waiting may
show little wall-clock benefit even when CPU work improves. The starter greetings
are too short and unrepresentative to estimate an application-level gain.

### Sampling and relinking

[Propeller](https://github.com/google/llvm-propeller) additionally needs compatible
Clang/LLD, a binary with basic-block address maps, hardware branch sampling and a
profile converter producing compiler and linker layout profiles. An LLVM backend
or LTO alone does not establish support. These templates do not ship an unverified
relinking preset; assess that separately on the intended OS, CPU and toolchain.
