---
name: refactor
description: 'Surgical, language-agnostic code refactoring to improve maintainability without changing behavior. Covers extracting functions, renaming, breaking down god functions/classes, improving type safety, eliminating code smells, applying design patterns, and characterization-testing legacy code before changing it. Works across any language and framework, with dedicated idiom notes for Python, JavaScript/TypeScript, Java, C#/.NET, Go, Rust, Ruby, PHP, C++, Kotlin, Swift, Groovy, Perl, PowerShell, Bash/shell, HTML/CSS, and frameworks including React, React Native, Angular, Node.js, and Spring Boot. Use this skill whenever the user asks to "clean up", "refactor", "improve", "tidy", "restructure", or "make more maintainable" any code, in any language or framework, even if they do not say the word "refactor" explicitly. Less drastic than a full rewrite; use for gradual improvements.'
---

# Code Refactor

> **Read this entire file before refactoring.** It is ~500 lines. The mandatory response structure ("Output Format") and the per-language idiom notes appear near the top, but the full smell catalog, examples, design-pattern guidance, and reference-file pointers continue to the end. If your file-reading tool loads files in chunks, continue past the first chunk — do not stop early.

## Overview

Improve code structure and readability **without changing external behavior**. Refactoring is gradual evolution, not revolution. Use this for improving existing code, not rewriting from scratch.

This skill is built on **language-agnostic principles** — the smells, fixes, and process apply to every language, and the code examples use pseudocode-like notation so they translate directly. On that foundation it carries **dedicated idiom notes** for a broad set of languages and frameworks: how you create a value object, enforce types, or run tests in each. The per-language mechanics are summarized in the "Per-Language Quick Notes" section below and detailed in `references/language-notes.md`. Framework-specific smells (React, React Native, Angular, Node.js, Spring Boot, .NET) live in `references/framework-notes.md`, loaded only when a framework request comes in.

## Enterprise Usage

This is a shared, company-wide skill. The universal refactoring principles — preserve behavior, work in small steps, never mix a behavior change into a refactor — are in "The Golden Rules" below and apply here too. On top of those, a few standing rules specific to safe team use:

- **Never refactor untested legacy code without a safety net first.** Establish characterization tests (see "Establishing a Safety Net") before touching behavior. This is non-negotiable for shared/production code.
- **Follow existing house conventions.** Match the surrounding codebase's style, naming, and patterns rather than introducing new ones mid-refactor. When a team has a documented standard, that standard wins over this skill's defaults.
- **Flag, don't assume.** If a refactor would touch a public API, shared contract, or cross-team boundary, call it out for review rather than proceeding silently.

## When to Use

- Code is hard to understand or maintain
- Functions/classes/modules are too large or do too much
- Code smells need addressing (duplication, magic values, deep nesting, etc.)
- Adding features is difficult because of the current structure
- User asks to "clean up", "refactor", "improve", "tidy", or "restructure" code

## How to Use This Skill

1. **Confirm the language, framework, versions, and test situation first.** Refactoring without a safety net is just editing. See "Establishing a Safety Net" below, and check versions per "Version Compatibility" below.
2. Identify the smell(s) using "Common Code Smells & Fixes".
3. Apply the matching fix in small steps, re-running tests after each.
4. Use the per-language notes for syntax/idiom specifics — never exceeding the detected language/framework version.
5. Verify against the checklist.
6. Present the result using the fixed structure in "Output Format" (immediately below).

---

## Version Compatibility

Before applying any idiom, determine **two independent versions** and never use a feature newer than either:

1. **Language version** — e.g. Java 8 vs 17, Python 3.8 vs 3.12, C# 7 vs 12, TypeScript target.
2. **Framework version** — e.g. Spring Boot 2 vs 3, React 16 vs 19, Angular 14 vs 17, .NET Framework 4.8 vs .NET 8, Node 14 vs 22.

These move independently: a project can be on a modern language but an older framework (e.g. **Java 17 + Spring Boot 2.7**), so both ceilings must be respected at once.

**Detect the version from project signals when available:**

- Java: `pom.xml` (`<maven.compiler.source>`, `spring-boot-starter-parent` version) or `build.gradle` (`sourceCompatibility`, dependency versions)
- C# / .NET: `.csproj` `<TargetFramework>` and NuGet package versions
- JS/TS: `package.json` (`engines`, dependency versions for react/@angular/core/next), `tsconfig.json` `target`
- Python: `pyproject.toml` / `setup.py` `python_requires`, or syntax already in the file
- Others: the lockfile/manifest, or idioms already present in the code

**Rules:**

- Never introduce a feature newer than the detected version (no `record` on Java 8, no hooks on React 15, no Jakarta/`@if` on Spring Boot 2 or Angular 14, no minimal APIs on .NET Framework).
- If a version cannot be determined and the refactor depends on a version-specific idiom, **ask** rather than guess.
- **Always state the assumed language and framework versions in the output** (in the Verification & Risks section), so a reviewer can catch a wrong assumption — e.g. "Assuming Java 17 + Spring Boot 2.7; using modern Java but avoiding Spring Boot 3 / Jakarta idioms."

---

## Output Format

Every refactoring response MUST follow this exact five-part structure, in this order. Do not omit a section; if one does not apply, write "None" under it rather than dropping it.

**1. Code Analysis (before refactoring)** — what the code currently does, and the specific smells found with their locations (e.g. "Long method — `processOrder`, 180 lines"; "Magic number — `0.1` in `applyDiscount`"). This is diagnosis only — do not introduce fixes here.

**2. Refactoring Summary** — one or two sentences stating what was changed and why.

**3. Changes Made (key differences)** — a **two-column table** of the meaningful changes, one row per change. The left column headed `Before` describes the prior state (the smell and where it lived); the right column headed `After` describes the new state. Use **plain words only — no code snippets and no complete file**, since the applied edits are already visible as a diff in the editor. Keep each cell to a short phrase anchored to a location (function / class / file). Do **not** name the formal refactoring operation or its benefit here — that is section 4's job, and repeating it is the overlap to avoid.

Example:

| Before                                                  | After                               |
| ------------------------------------------------------- | ----------------------------------- |
| Three-branch shipping `if`-chain in `calculateShipping` | Lookup keyed by method              |
| Active/inactive user blocks inline in `buildReport`     | Shared `printUserSection` extracted |
| Eight positional `createUser` parameters                | Single `UserData` object            |

**4. Techniques & Impact** — each refactoring applied, named by its standard operation (e.g. "Extract Method", "Introduce Parameter Object", "Constructor injection", "Replace Nested Conditional with Guard Clauses"), paired with its concrete benefit (what it improves and why it matters). Refer to each change by its technique and benefit — do not re-list the locations already given in section 3.

**5. Verification & Risks** — the assumed language and framework versions; the test situation before changes (existing tests found / characterization tests added / **none exist → explicit warning that the refactor is unverified and tests should be added first**), how behavior was confirmed unchanged (tests pass / characterization diff clean / manual reasoning), and anything a reviewer must check by hand — public-API or cross-team impact, and any further refactoring deliberately deferred.

Rules for the template:

- Never mix a behavior change into a refactoring response. If a bug fix or feature is in scope, stop and flag it under Verification & Risks as a separate task — do not silently bundle it in.
- Keep each section tight and scannable — this is a review aid, not an essay. No code goes in the response at all; the applied edits are visible as a diff in the editor, and the five sections exist to explain what the diff cannot — the diagnosis, intent, techniques, and risks.
- If the request is too large for one safe step, cover only the first unit and list the remaining units under Verification & Risks, rather than attempting everything at once.

---

## Per-Language Quick Notes

Mechanics differ by language; the refactorings don't. Highlights here; full details in `references/language-notes.md`.

- **Python** — type hints + mypy/pyright for type safety; `@dataclass`/`NamedTuple` for parameter objects; `enum.Enum` for type codes; `pytest` + snapshot/`approvaltests` for characterization tests; `vulture`/`ruff` for dead code.
- **JavaScript / TypeScript** — prefer TS for type safety (unions, `interface`, generics, `strictNullChecks`); object params for long lists; jest/vitest snapshots for characterization; `ts-prune`/ESLint for dead code.
- **Java** — `record` for parameter objects/value types; `enum` for type codes; interfaces + strategy; JUnit + ApprovalTests; IDE inspections for dead code.
- **C#** — `record`/`readonly struct` for value types; `enum`; pattern matching to flatten conditionals; xUnit/NUnit + ApprovalTests.
- **Go** — small interfaces for strategy; structs for parameter objects; explicit error returns instead of deep nesting; golden files for characterization; `deadcode`/`staticcheck`.
- **Rust** — `enum` + `match` (often replaces strategy outright); `Result`/`Option` for control flow; newtypes for primitive obsession; `insta` for snapshot tests; `cargo clippy`.
- **Ruby** — Plain Old Ruby Objects per responsibility; keyword args / value objects for long params; Sorbet/RBS for gradual typing; `rspec` snapshots.
- **PHP** — typed properties, enums (8.1+), readonly classes for value objects; PHPUnit + snapshot tests.
- **C++** — `enum class`, strong typedefs/value classes; `std::variant` + `std::visit` for strategy-like dispatch; ApprovalTests.cpp; clang-tidy for smells.
- **Kotlin / Swift** — `data class`/`struct` for parameter objects; sealed classes/enums + `when`/`switch` for polymorphic dispatch; `Result` for control flow.
- **Groovy** — `@Immutable`/`@Canonical` for value objects; closures as strategies; `@CompileStatic` for gradual typing; Spock + CodeNarc.
- **Perl** — `use strict; use warnings`; extract subs; `use constant`/`Readonly` for magic values; hash of coderefs for dispatch; Perl::Critic + Test::More.
- **Shell (Bash/POSIX)** — `set -euo pipefail`; extract functions; quote variables; `readonly` constants; shellcheck + bats.
- **PowerShell** — functions with `[CmdletBinding()]` + typed `param()`; approved Verb-Noun names; splatting for long params; PSScriptAnalyzer + Pester.
- **HTML / CSS** — extract repeated markup into components/partials; semantic elements over div soup; CSS custom properties for repeated values; BEM/utility naming; remove dead CSS (PurgeCSS); verify with visual-regression tests, not unit tests.

## Framework-Specific Refactoring

Frameworks add their own smells on top of the base language. The agent loads `references/framework-notes.md` only when a framework request comes in. Highlights:

- **React / React Native** — extract child components and **custom hooks** to kill duplication; replace prop drilling with Context; split overloaded `useEffect`s; compute derived values in render instead of storing them. RN adds: extract `StyleSheet`, split `.ios`/`.android` files, memoize list rows.
- **Angular** — move logic out of components into injectable **services**; replace manual `subscribe()` with the **async pipe** and RxJS operators; lift shared state into services/stores; `OnPush`/Signals for change detection; strategy via DI tokens.
- **Node.js** — convert callback hell to `async/await`; split fat route handlers into **handler → service → repository**; keep logic out of `req`/`res` so it's testable; centralize error handling.
- **Spring Boot** — thin controllers, logic in `@Service`; **constructor injection** over field injection; `@ConfigurationProperties` typed config; strategy beans instead of type-switches.
- **.NET / ASP.NET Core** — thin controllers with DI; typed `IOptions<T>` config; `record` DTOs; cross-cutting concerns in middleware; extract inline minimal-API handlers.

---

## Refactoring Principles

### The Golden Rules

1. **Behavior is preserved** — refactoring changes _how_, never _what_.
2. **Small steps** — make tiny changes, verify after each.
3. **Version control is your friend** — commit at every green (passing) state.
4. **Tests are essential** — without a way to verify behavior, you are editing, not refactoring.
5. **One thing at a time** — never mix refactoring with feature or behavior changes in the same commit.

### When NOT to Refactor

- Code that works and will never change again ("if it ain't broke…").
- Critical code with no tests — **add a safety net first** (see below).
- Under a tight deadline with no buffer for verification.
- "Just because" — refactoring needs a concrete purpose (readability for an upcoming change, removing a blocker, fixing a smell that caused a bug).

---

## Establishing a Safety Net (do this BEFORE touching legacy code)

You cannot preserve behavior you haven't pinned down. If the code lacks tests, add **characterization tests** (a.k.a. golden-master / approval tests) first. These don't assert "correct" behavior — they assert _current_ behavior, so any accidental change shows up as a failure.

Process, in any language:

1. Pick the unit you're about to refactor (a function, class, or module).
2. Feed it representative and edge-case inputs.
3. Capture whatever it currently returns/prints/writes as the "expected" value — even if that output looks wrong. You're freezing today's behavior.
4. Now refactor. If a characterization test fails, you changed behavior — investigate before continuing.
5. Once refactoring is done, you may _then_ fix genuine bugs as separate, clearly-labeled commits with their own assertions.

Pseudocode of the idea:

```
# Characterization test — pins CURRENT behavior, not "correct" behavior
result = legacyFunction(sampleInput)
assertEquals(result, "<whatever it produces today>")   # capture, don't judge
```

Approval-test tooling exists in most ecosystems (e.g. snapshot tests in JS/TS, `pytest`'s snapshot plugins or `approvaltests` in Python, ApprovalTests in Java/C#/C++, golden files in Go). Use them when output is large or structured.

---

## Common Code Smells & Fixes

Each fix below is shown in language-neutral pseudocode. The transformation is what matters; adapt syntax to your language.

### 1. Long Method / Function

A function that does many things is hard to read, test, and reuse. Extract each responsibility into its own well-named function.

```
# BEFORE: one function doing everything
function processOrder(orderId):
    # fetch order ...
    # validate order ...
    # calculate pricing ...
    # update inventory ...
    # create shipment ...
    # send notifications ...

# AFTER: a coordinator delegating to focused functions
function processOrder(orderId):
    order    = fetchOrder(orderId)
    validateOrder(order)
    pricing  = calculatePricing(order)
    updateInventory(order)
    shipment = createShipment(order)
    sendNotifications(order, pricing, shipment)
    return { order, pricing, shipment }
```

### 2. Duplicated Code

The same logic in two places drifts apart over time. Extract it once.

```
# BEFORE: rate logic duplicated
function calculateUserDiscount(user):
    if user.membership == "gold":   return user.total * 0.2
    if user.membership == "silver": return user.total * 0.1
    return 0

function calculateOrderDiscount(order):
    if order.user.membership == "gold":   return order.total * 0.2
    if order.user.membership == "silver": return order.total * 0.1
    return 0

# AFTER: single source of truth
function membershipDiscountRate(membership):
    rates = { "gold": 0.2, "silver": 0.1 }
    return rates.get(membership, 0)

function calculateUserDiscount(user):
    return user.total * membershipDiscountRate(user.membership)

function calculateOrderDiscount(order):
    return order.total * membershipDiscountRate(order.user.membership)
```

### 3. Large Class / Module (God Object)

A type that knows and does too much. Split by responsibility (Single Responsibility Principle).

```
# BEFORE: one class owns everything
class UserManager:
    createUser(); updateUser(); deleteUser()
    sendEmail(); generateReport(); handlePayment(); validateAddress()
    # ...many more

# AFTER: one responsibility per type
class UserService:    create(data); update(id, data); delete(id)
class EmailService:   send(to, subject, body)
class ReportService:  generate(type, params)
class PaymentService: process(amount, method)
```

### 4. Long Parameter List

Too many parameters are hard to call correctly and easy to mis-order. Group related ones into an object/struct/record.

```
# BEFORE
function createUser(email, password, name, age, address, city, country, phone): ...

# AFTER: a parameter object / struct / record / data class
type UserData = {
    email, password, name,
    age?, address?, phone?
}
function createUser(data: UserData): ...

# When construction is complex, a builder reads well in any OO language:
user = UserBuilder()
    .email("test@example.com")
    .password("secure123")
    .name("Test User")
    .address(address)
    .build()
```

### 5. Feature Envy

A method uses another object's data more than its own. Move the logic to the object that owns the data.

```
# BEFORE: Order reaches into User's fields to decide a rate
class Order:
    calculateDiscount(user):
        if user.membershipLevel == "gold": return this.total * 0.2
        if user.accountAge > 365:          return this.total * 0.1
        return 0

# AFTER: User owns the rate; Order just applies it
class User:
    discountRate():
        if this.membershipLevel == "gold": return 0.2
        if this.accountAge > 365:          return 0.1
        return 0

class Order:
    calculateDiscount(user):
        return this.total * user.discountRate()
```

### 6. Primitive Obsession

Using raw strings/numbers for domain concepts scatters validation and invites invalid states. Wrap them in small value types that validate on construction.

```
# BEFORE: a bare string is "an email" only by convention
function sendEmail(to, subject, body): ...
sendEmail("user@example.com", "Hello", "...")

# AFTER: a value type that cannot exist in an invalid state
class Email:
    constructor(value):
        if not Email.isValid(value): raise Error("Invalid email")
        this.value = value
    static isValid(v): return matches(v, /^[^\s@]+@[^\s@]+\.[^\s@]+$/)

class PhoneNumber:
    constructor(country, number):
        if not PhoneNumber.isValid(country, number): raise Error("Invalid phone")
        this.country = country; this.number = number
    toString(): return country + "-" + number

email = Email("user@example.com")
phone = PhoneNumber("1", "555-1234")
```

> The regex above is illustrative only — for real validation use a vetted library or your platform's built-in validator rather than a hand-rolled pattern.

### 7. Magic Numbers / Strings

Unexplained literals hide intent. Name them.

```
# BEFORE
if user.status == 2: ...
discount = total * 0.15
sleep(86400000)

# AFTER: named constants / enums
enum UserStatus { ACTIVE = 1, INACTIVE = 2, SUSPENDED = 3 }
constant DISCOUNT_PREMIUM = 0.15
constant ONE_DAY_MS = 24 * 60 * 60 * 1000

if user.status == UserStatus.INACTIVE: ...
discount = total * DISCOUNT_PREMIUM
sleep(ONE_DAY_MS)
```

### 8. Nested Conditionals ("Arrow Code")

Deeply indented `if`s are hard to follow. Flatten with guard clauses / early returns.

```
# BEFORE: deeply nested, hard to track which branch you're in
function process(order):
    if order:
        if order.user:
            if order.user.isActive:
                if order.total > 0:
                    return processOrder(order)
                else:
                    return { error: "Invalid total" }
            else:
                return { error: "User inactive" }
        else:
            return { error: "No user" }
    else:
        return { error: "No order" }

# AFTER: guard clauses, one happy path at the end
function process(order):
    if not order:               return { error: "No order" }
    if not order.user:          return { error: "No user" }
    if not order.user.isActive: return { error: "User inactive" }
    if order.total <= 0:        return { error: "Invalid total" }
    return processOrder(order)
```

If your language has a result/option type (Rust `Result`, Kotlin/Swift `Result`, functional `Either`), chaining validators is an even cleaner option — see `references/language-notes.md`.

### 9. Dead Code

Unused functions, variables, imports, and commented-out blocks add noise and mislead readers.

```
# BEFORE
function oldImplementation(): ...    # never called
constant DEPRECATED_VALUE = 5        # never read
import unusedThing
# function oldCode(): ...            # commented out

# AFTER: delete it. Version control is the archive.
```

Use your language's tooling to find dead code (linters, `--dead-code` flags, IDE "unused symbol" warnings, coverage reports).

### 10. Inappropriate Intimacy (Train Wrecks)

One object reaches through several others (`a.b.c.d`), coupling itself to internal structure. Tell objects what to do; don't pull out their internals (Law of Demeter).

```
# BEFORE: reaching deep into the object graph
class OrderProcessor:
    process(order):
        street = order.user.profile.address.street    # too intimate
        cfg    = order.repository.connection.config    # breaks encapsulation

# AFTER: ask the object to do the work
class OrderProcessor:
    process(order):
        order.shippingAddress()   # Order knows how to get it
        order.save()              # Order knows how to save itself
```

---

## Introducing Type Safety

Where a language supports it, replace loosely-typed values with explicit types and constrained domains — this turns whole classes of bugs into compile-time (or lint-time) errors. Replace stringly-typed inputs (e.g. a `membership` string driving discount logic) with a constrained type (union/enum), return a structured result type instead of a bare number, and validate at boundaries.

In statically typed languages use the type system directly (unions/enums, records/structs, generics, nullability annotations). In dynamically typed languages, approximate with gradual typing (Python type hints + mypy/pyright, TypeScript, Sorbet for Ruby, PHP typed properties) and runtime validation at boundaries. The "Gradual / Static Typing" table in `references/language-notes.md` gives the exact mechanism per language.

---

## Design Patterns for Refactoring

Patterns are tools, not goals — reach for one only when it removes a real smell. The five most common refactoring targets, each with the smell it fixes:

- **Strategy** — replace a type-switch on behavior (e.g. branching on a `method` string) with interchangeable objects behind a common interface.
- **Chain of Responsibility** — turn a function accumulating many independent checks into a pipeline where each check is its own link (or, more simply, a list of validator functions run in turn).
- **Factory / Factory Method** — centralize messy object construction scattered across the codebase.
- **Observer** — decouple "something changed" from "everyone who cares", replacing direct cross-calls.
- **Decorator** — add behavior (logging, caching, retries) without subclass explosions or editing the core type.

Full before/after implementations for each — in multiple languages, with the functional vs OO trade-offs and guidance on when to prefer each — are in `references/design-patterns.md`. Load it whenever a refactor calls for a pattern.

---

## The Safe Refactoring Process

```
1. PREPARE
   - Ensure a safety net exists (tests or characterization tests). Add if missing.
   - Commit the current state. Work on a branch.

2. IDENTIFY
   - Find the specific smell. Understand current behavior. Plan the change.

3. REFACTOR (small steps)
   - Make ONE small change → run tests → commit if green → repeat.

4. VERIFY
   - All tests pass. Manual/exploratory check if needed. Performance not regressed.

5. CLEAN UP
   - Update comments and docs. Final commit. Open PR.
```

---

## Refactoring Checklist

### Safety Net

- [ ] Tests (or characterization tests) exist and pass before starting
- [ ] Behavior is verified unchanged after each step

### Code Quality

- [ ] Functions are small and do one thing
- [ ] No duplicated logic
- [ ] Descriptive names for variables, functions, types
- [ ] No magic numbers/strings
- [ ] Dead code removed

### Structure

- [ ] Related code lives together; clear module boundaries
- [ ] Dependencies flow in one direction; no circular dependencies
- [ ] No train wrecks / Law of Demeter respected

### Type Safety (where the language supports it)

- [ ] Public APIs have explicit types
- [ ] No escape-hatch types (`any`, `Object`, `interface{}`, `void*`) without justification
- [ ] Nullability is explicit

---

## Common Refactoring Operations

| Operation                                     | Description                                                  |
| --------------------------------------------- | ------------------------------------------------------------ |
| Extract Method/Function                       | Turn a code fragment into a named callable                   |
| Extract Class/Module                          | Move related behavior into a new type                        |
| Extract Interface                             | Define an interface from an implementation                   |
| Inline Method/Class                           | Move a body back to its caller when indirection adds nothing |
| Pull Up / Push Down Member                    | Move a member up to a base or down to a subtype              |
| Rename                                        | Improve clarity of any symbol                                |
| Introduce Parameter Object                    | Group related parameters                                     |
| Replace Conditional with Polymorphism         | Use types/strategy instead of switch/if                      |
| Replace Magic Number with Constant            | Name the value                                               |
| Decompose / Consolidate Conditional           | Break up or combine complex conditions                       |
| Replace Nested Conditional with Guard Clauses | Early returns                                                |
| Introduce Null Object                         | Remove repetitive null checks                                |
| Replace Type Code with Class/Enum             | Strong typing for categories                                 |
| Replace Inheritance with Delegation           | Composition over inheritance                                 |

---

## Reference Files

- `references/language-notes.md` — idiomatic mechanics for each smell/fix per language, including shell scripting (Bash/PowerShell/Perl) and markup/styling (HTML/CSS).
- `references/framework-notes.md` — framework-specific smells and fixes for React, React Native, Angular, Node.js, Spring Boot, and .NET / ASP.NET Core.
- `references/design-patterns.md` — Strategy, Chain of Responsibility, Factory, Observer, and Decorator, each with before/after in multiple languages, plus functional alternatives.
