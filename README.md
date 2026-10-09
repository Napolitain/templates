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

```sh
git clone --recurse-submodules https://github.com/Napolitain/templates
git submodule update --remote    # move every submodule to its latest main
```
