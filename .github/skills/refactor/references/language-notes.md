# Language Notes

Idiomatic mechanics for applying each refactoring per language. The *refactoring* is the same everywhere; only the syntax and tooling change. Use this when you know the smell and the fix but need the language-correct way to express it.

## Table of Contents
- [Parameter Objects / Value Types](#parameter-objects--value-types)
- [Type Codes → Enums](#type-codes--enums)
- [Strategy / Polymorphic Dispatch](#strategy--polymorphic-dispatch)
- [Flattening Conditionals & Result Types](#flattening-conditionals--result-types)
- [Gradual / Static Typing](#gradual--static-typing)
- [Characterization & Snapshot Testing](#characterization--snapshot-testing)
- [Finding Dead Code](#finding-dead-code)
- [Shell & Scripting (Bash / Shell, PowerShell, Perl)](#shell--scripting-bash--shell-powershell-perl)
- [Markup & Styling (HTML, CSS)](#markup--styling-html--css)

---

## Parameter Objects / Value Types

| Language | Idiom |
| --- | --- |
| Python | `@dataclass(frozen=True)`, `typing.NamedTuple`, or Pydantic models |
| TypeScript | `interface`/`type` for params; `class` with private ctor for validated value objects |
| Java | `record UserData(...)`; for validation, a record with a compact canonical constructor |
| C# | `record`, `readonly record struct` |
| Go | a `struct`; validate in a `NewX` constructor function returning `(X, error)` |
| Rust | a `struct`; newtype pattern `struct Email(String)` with a `try_from`/`new` returning `Result` |
| Ruby | a small PORO (`Struct.new` or a class) with keyword args |
| PHP | `readonly class` (8.2+) or class with typed promoted properties |
| C++ | a `struct`/class; constructor validates and throws or use a factory returning `std::optional`/`expected` |
| Kotlin | `data class`; `require(...)` in `init` for validation; `@JvmInline value class` for newtypes |
| Swift | `struct`; failable initializer `init?` or throwing `init` for validation |
| Groovy | `@Immutable`/`@Canonical` AST transforms; map-based constructors for long lists |
| Perl | a hashref of named args (`my %args = @_;`) or a small blessed class / `Object::Pad` |
| PowerShell | `[PSCustomObject]@{...}` or a `class`; splatting (`@params`) to pass a param bundle |

## Type Codes → Enums

| Language | Idiom |
| --- | --- |
| Python | `enum.Enum` / `enum.IntEnum` |
| TypeScript | string-literal union (`type Status = "active" \| "inactive"`) preferred over numeric `enum` |
| Java / C# / Kotlin / Swift | native `enum` (can carry methods/associated values) |
| Go | typed constants with `iota` |
| Rust | `enum` (can carry data per variant) |
| Ruby | frozen constants or a small enum gem |
| PHP | native `enum` (8.1+), backed or pure |
| C++ | `enum class` |
| Groovy | `enum` (full Java enum semantics) |
| Perl | `use constant { ... }` or a `Readonly` hash; dispatch on the constant |
| PowerShell | `enum Status { Active; Inactive }` (PS 5+) |

## Strategy / Polymorphic Dispatch

- **OO languages (Java, C#, TS, Python, Ruby, Kotlin, Swift, PHP):** define an interface/abstract type with one method; one concrete class per strategy; inject the chosen strategy.
- **Go:** define a small interface; any type with the method satisfies it implicitly.
- **Rust:** often skip the pattern — a `match` on an `enum` is idiomatic. Use `Box<dyn Trait>` when strategies must be chosen at runtime/extensible.
- **C++:** an abstract base class with a virtual method, or `std::variant` + `std::visit`, or `std::function` members.
- **Functional style (any language with first-class functions):** a strategy is just a function; store functions in a map keyed by case.

```
# functional strategy in any language with first-class functions / closures
shippingStrategies = {
    "standard":  order -> order.total > 50  ? 0 : 5.99,
    "express":   order -> order.total > 100 ? 9.99 : 14.99,
    "overnight": order -> 29.99,
}
cost = shippingStrategies[method](order)
```

## Flattening Conditionals & Result Types

Guard clauses work in every language. Beyond that:

- **Rust:** `Result<T, E>` / `Option<T>` with the `?` operator for early exit; combinators `map`, `and_then`.
- **Kotlin:** `Result<T>`, `?.`, `?:`, `requireNotNull`; `when` for multi-branch.
- **Swift:** `guard let ... else { return }`, `Result`, optionals.
- **Go:** the `if err != nil { return ... }` early-return idiom is the guard clause.
- **TypeScript / JS:** early `return`; libraries like `neverthrow` for a `Result` type.
- **Python:** early `return`/`raise`; `match` (3.10+) for multi-branch dispatch.
- **Functional (Either/Result):** chain validators with `flatMap`/`andThen` so the first failure short-circuits.

## Gradual / Static Typing

| Language | Mechanism |
| --- | --- |
| Python | type hints + `mypy` or `pyright`; `typing.Protocol` for structural typing |
| Ruby | Sorbet (`sig`) or RBS files |
| JS → TS | migrate files to `.ts`; enable `strict` (esp. `strictNullChecks`) |
| PHP | typed properties, return types, `declare(strict_types=1)` |
| Already static (Java, C#, Go, Rust, C++, Kotlin, Swift) | use unions/generics/nullability features the language already provides |

Validate at boundaries (API inputs, file/DB reads) where the type system can't reach.

## Characterization & Snapshot Testing

| Language | Tooling |
| --- | --- |
| Python | `pytest` + `syrupy`/`pytest-snapshot`, or `approvaltests` |
| JS/TS | jest/vitest `toMatchSnapshot`, or `approvals` |
| Java | JUnit + ApprovalTests (Java) |
| C# | xUnit/NUnit + ApprovalTests.Net / Verify |
| Go | golden files (`-update` flag pattern), `goldie`, or `cupaloy` |
| Rust | `insta` |
| Ruby | RSpec snapshot gems / `approvals` |
| PHP | PHPUnit + spatie/phpunit-snapshot-assertions |
| C++ | ApprovalTests.cpp |
| Kotlin/Swift | same JVM/Apple tooling (Approval/Verify, snapshot-testing libs) |

Capture *current* output as the baseline, then refactor; a diff means behavior changed.

## Finding Dead Code

| Language | Tool |
| --- | --- |
| Python | `vulture`, `ruff --select F401` (unused imports), coverage |
| JS/TS | `ts-prune`, ESLint `no-unused-vars`, `knip` |
| Java | IDE inspections, `pmd`, `error-prone` |
| C# | Roslyn analyzers, `dotnet format` |
| Go | `deadcode`, `staticcheck`, `go vet` |
| Rust | compiler `dead_code` warnings, `cargo clippy` |
| C++ | `clang-tidy`, `-Wunused`, cppcheck |
| Ruby | `debride`, RuboCop |
| PHP | PHPStan, Psalm (`UnusedMethod`/`UnusedVariable`) |

Always confirm a symbol is truly unused (reflection, dynamic dispatch, public API, plugins) before deleting.

---

## Shell & Scripting (Bash / Shell, PowerShell, Perl)

These drift into unmaintainable "big ball of script" quickly; the same smells (long function, duplication, magic values, deep nesting) all apply.

- **Bash / POSIX shell** — put `set -euo pipefail` at the top so failures surface; **extract functions** for repeated command pipelines; always **quote variables** (`"$var"`); replace magic paths/flags with `readonly` named constants; guard clauses via early `return`/`exit`; dispatch tables via `case` or an associative array of function names. Lint with **shellcheck**; characterize with **bats** (Bash Automated Testing System) capturing stdout/exit codes.
- **PowerShell** — extract logic into functions/cmdlets with `[CmdletBinding()]` and a typed `param()` block; use **approved Verb-Noun** names (`Get-`, `Set-`, `New-`); pass objects, don't parse text; **splatting** (`@params`) is the parameter-object idiom for long arg lists; `enum`/classes (PS5+) for type codes and value objects. Lint with **PSScriptAnalyzer**; test with **Pester** (supports snapshot-style assertions).
- **Perl** — always `use strict; use warnings`; extract repeated logic into **subs**; named constants via `use constant` or `Readonly`; a **hash of coderefs** is the idiomatic strategy/dispatch table; gradual structure via `Object::Pad`/`Moo`/`Moose` classes for value objects. Lint with **Perl::Critic**; test with **Test::More** + **Test::Snapshot** for characterization.

---

## Markup & Styling (HTML / CSS)

HTML and CSS aren't behavioral code, so most "method" refactorings don't apply. But duplication, dead code, and poor structure absolutely do. Here "behavior preserved" means **the rendered output looks and works the same** — verify with **visual-regression tests** (Playwright/Percy/Chromatic screenshots) instead of unit tests.

- **HTML** — extract repeated markup into reusable **components / partials / includes / web components** (this is "remove duplication" + "extract method" for templates); use **semantic elements** instead of `div` soup; pull inline `style=`/`onclick=` out into CSS/JS; reduce wrapper nesting. Validate structure with the W3C validator / `html-validate`.
- **CSS** — replace repeated values (colors, spacing) with **custom properties** (`--brand: ...`); consolidate duplicated rule sets; adopt a naming convention (**BEM** or utility classes) to tame specificity wars; replace fragile deep descendant selectors with single-class selectors; remove **dead/unused CSS** (Chrome DevTools Coverage, **PurgeCSS**); lint with **stylelint**.
