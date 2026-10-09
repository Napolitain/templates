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
