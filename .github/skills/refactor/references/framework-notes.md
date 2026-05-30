# Framework Notes

Frameworks inherit their base language's refactorings (React/React Native/Node.js/Angular → see the JavaScript/TypeScript notes; Spring Boot → Java/Kotlin; .NET → C#) but add their *own* smells. "Behavior preserved" means the user-visible behavior and public contract are unchanged; verify with the framework's standard test tooling.

## Table of Contents
- [React](#react)
- [React Native](#react-native)
- [Angular](#angular)
- [Node.js](#nodejs)
- [Spring Boot](#spring-boot)
- [.NET / ASP.NET Core](#net--aspnet-core)

---

## React

| Smell | Fix |
| --- | --- |
| Giant component (hundreds of lines, many concerns) | **Extract child components**; separate presentational ("dumb") from container ("smart") components |
| Same stateful logic duplicated across components | **Extract a custom hook** (`useThing()`) — this is "extract method" for React |
| Prop drilling through many layers | Lift to **Context**, or use composition (`children`) instead of threading props |
| One `useEffect` doing several unrelated things | Split into **focused effects**; fix dependency arrays so each effect has one job |
| Derived value stored in state (and kept in sync by hand) | **Compute it during render** instead of storing it |
| Deeply nested conditional JSX | Extract to a function/component; use early returns for loading/error/empty states |
| Handlers/objects rebuilt every render in a hot path | `useCallback`/`useMemo` — **only after measuring**, not by default |

Safety net: **React Testing Library** (assert behavior, not implementation) + snapshot tests. Keep the same rendered output and interactions through the refactor.

## React Native

All the React refactors above apply. Additional RN-specific smells:

| Smell | Fix |
| --- | --- |
| `StyleSheet`/inline styles tangled into render | Extract styles into a `StyleSheet.create({...})` block outside the component |
| Inline `Platform.OS === 'ios' ? ... : ...` branching everywhere | Split into `Component.ios.tsx` / `Component.android.tsx` files |
| `FlatList` re-rendering all rows | Extract a memoized item component (`React.memo`); stable `keyExtractor` |
| Business logic mixed into screens | Push into hooks/services so screens stay thin and testable |

Test with React Native Testing Library; consider golden/visual tests for layout-sensitive screens.

## Angular

(Base language: TypeScript — see the TS rows in language-notes.md.)

| Smell | Fix |
| --- | --- |
| Fat component with business logic and HTTP calls inside | Move logic into an injectable **`@Injectable()` service**; component only binds view state and delegates |
| Logic in the template or in lifecycle hooks doing too much | Extract to service methods; keep `ngOnInit` thin; split one overloaded hook into focused methods |
| Manual `subscribe()` everywhere (and leaked subscriptions) | Prefer the **`async` pipe** in the template; otherwise unsubscribe via `takeUntilDestroyed`/`takeUntil` so streams are cleaned up |
| Imperative nested subscriptions (subscribe inside subscribe) | Compose with **RxJS operators** (`switchMap`, `combineLatest`, `map`) instead of nesting |
| Repeated markup across components | Extract a shared **presentational component**; pass data via `@Input()`, emit via `@Output()` |
| Prop/`@Input()` drilling or shared mutable state | Lift shared state into a service (or a store like NgRx/Signals); inject where needed |
| Heavy work re-running every change-detection cycle | `OnPush` change detection; pure pipes; **Signals** (Angular 16+) for fine-grained reactivity |
| Magic strings for routes/events/config | Named constants, typed route enums, `InjectionToken` for config (the parameter-object idiom for DI) |
| Logic branching on a type code | Strategy via DI: an interface + multiple providers, selected with an `InjectionToken`/factory |

Safety net: **Jasmine/Karma** or **Jest** with `TestBed`; `HttpTestingController` for HTTP; component harnesses for UI. Assert behavior and rendered output, not internal change-detection details.

## Node.js

| Smell | Fix |
| --- | --- |
| Callback hell / pyramid of doom | Convert to **`async`/`await`** (promisify callback APIs first) |
| Fat Express/Fastify route handlers with business logic inside | Split into layers: thin **handler → service → repository**; handler only parses input and formats output |
| Logic coupled to `req`/`res` (untestable without HTTP) | Move logic into **pure functions/services** that take plain data and return data |
| Inconsistent / swallowed errors | Centralized **error-handling middleware**; typed errors or a `Result`-style return; never leave promises unhandled |
| Config and secrets read ad hoc throughout | Centralize config; pass dependencies in (dependency injection) rather than importing singletons everywhere |
| Magic strings for routes/status/events | Named constants / enums (TS) |

Safety net: **jest/vitest** for units, **supertest** for endpoints; `knip`/`ts-prune` for dead code; ESLint for smells.

## Spring Boot

(Base language: Java or Kotlin — see those notes too.)

| Smell | Fix |
| --- | --- |
| Fat `@RestController` with business logic | Move logic to an `@Service`; controller only maps DTOs and delegates |
| Field injection (`@Autowired` on a field) | **Constructor injection** → fields become `final`, the class is immutable and unit-testable without Spring |
| God service class | Split by responsibility; extract domain classes (don't leave logic anemic) |
| Business rules living in `@Entity` or repository | Keep repositories persistence-only; rules go in the service/domain layer |
| Scattered magic config values | `@ConfigurationProperties` **typed config object** (a parameter object for settings) |
| `if (type.equals("x"))` branching on a kind | Define a strategy interface, make each impl a `@Component`, inject `List<Strategy>` or a `Map<String, Strategy>` and pick by key |

Safety net: **JUnit 5** + `MockMvc`/`@SpringBootTest`; **Testcontainers** to characterize behavior against a real DB; **ArchUnit** to enforce layer boundaries after you've cleaned them up.

## .NET / ASP.NET Core

(Base language: C# — see the C# rows in language-notes.md. Same patterns apply to F#/VB on the platform.)

| Smell | Fix |
| --- | --- |
| Fat controller / endpoint with business logic | Thin controller → inject a service via the built-in **DI container**; logic lives in the service |
| `new`-ing dependencies inside classes | Register in DI and inject via constructor (testable, swappable) |
| Config read from `IConfiguration` by string keys everywhere | Bind to a typed `IOptions<T>` **options class** (parameter object for config) |
| Long minimal-API lambdas inline in `Program.cs` | Extract endpoint handlers into named methods/classes |
| Cross-cutting concerns (logging, auth, errors) copy-pasted | Move into **middleware**/filters |
| DTOs as mutable classes with setters | Use `record` types for immutable DTOs |
| Branching on a type code | Strategy via DI: register implementations, resolve with keyed services or a factory |

Safety net: **xUnit/NUnit**; `WebApplicationFactory` for integration/characterization tests; Roslyn analyzers + `dotnet format` for dead code and style.
