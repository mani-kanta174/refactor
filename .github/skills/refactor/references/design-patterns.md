# Design Patterns for Refactoring

Patterns are a means to remove a specific smell, not an end. Apply one only when it makes the code simpler to change or read. Each pattern below shows the smell it cures, a before/after, and language-specific notes.

## Table of Contents
- [Strategy](#strategy)
- [Chain of Responsibility](#chain-of-responsibility)
- [Factory / Factory Method](#factory--factory-method)
- [Observer](#observer)
- [Decorator](#decorator)
- [Choosing Between Them](#choosing-between-them)

---

## Strategy

**Cures:** a `switch`/`if` chain that selects between interchangeable behaviors; the chain grows every time a new case appears.

```
# BEFORE
function calculateShipping(order, method):
    if method == "standard":  return order.total > 50  ? 0 : 5.99
    if method == "express":   return order.total > 100 ? 9.99 : 14.99
    if method == "overnight": return 29.99

# AFTER (OO)
interface ShippingStrategy:        calculate(order) -> number
class StandardShipping:  calculate(order): return order.total > 50  ? 0 : 5.99
class ExpressShipping:   calculate(order): return order.total > 100 ? 9.99 : 14.99
class OvernightShipping: calculate(order): return 29.99

function calculateShipping(order, strategy): return strategy.calculate(order)
```

**Functional alternative** (any language with first-class functions): store strategies in a map.

```
strategies = {
    "standard":  o -> o.total > 50  ? 0 : 5.99,
    "express":   o -> o.total > 100 ? 9.99 : 14.99,
    "overnight": o -> 29.99,
}
cost = strategies[method](order)
```

**Language notes:** Rust/Kotlin/Swift often express this as `match`/`when`/`switch` on an `enum` rather than separate classes. Go uses a small interface. C++ can use `std::function` members or `std::variant`.

---

## Chain of Responsibility

**Cures:** a long validation/processing function where each check is independent and the set of checks changes over time.

The OO form needs three things on the base class, declared once: the `next` link, a `setNext` that returns the link (so wiring chains read nicely), and a single `handle` entry point that runs this link's check then defers. Subclasses implement only their own `check`.

```
abstract class Validator:
    next = null
    setNext(v):
        this.next = v
        return v                         # return the link just added
    handle(user):
        error = this.check(user)         # this link's rule
        if error != null: return error
        if this.next != null: return this.next.handle(user)
        return null
    abstract check(user) -> string_or_null

class EmailRequiredValidator(Validator):
    check(u): return u.email ? null : "Email required"
class EmailFormatValidator(Validator):
    check(u): return (u.email and not isValidEmail(u.email)) ? "Invalid email" : null
class AgeValidator(Validator):
    check(u): return u.age < 18 ? "Must be 18+" : null

# Wire it up — setNext returns each link so the calls chain:
chain = EmailRequiredValidator()
chain.setNext(EmailFormatValidator()).setNext(AgeValidator())

firstError = chain.handle(user)          # null if all pass
```

**Functional alternative** — usually simpler in practice. A chain of independent checks is just a list of functions:

```
validators = [
    u -> u.email ? null : "Email required",
    u -> (u.email and not isValidEmail(u.email)) ? "Invalid email" : null,
    u -> u.age < 18 ? "Must be 18+" : null,
]

# first error (short-circuit):
firstError = firstNonNull(map(validators, v -> v(user)))

# OR collect all errors:
allErrors = filterNotNull(map(validators, v -> v(user)))
```

Prefer the functional version unless links must carry state or be reconfigured at runtime.

---

## Factory / Factory Method

**Cures:** object construction logic (which subtype to build, default wiring, validation) duplicated and scattered across callers.

```
# BEFORE: callers each decide how to build the right exporter
if format == "csv":  exporter = CsvExporter(); exporter.delimiter = ","
elif format == "tsv": exporter = CsvExporter(); exporter.delimiter = "\t"
elif format == "json": exporter = JsonExporter(pretty=true)

# AFTER: one place owns construction
function exporterFor(format) -> Exporter:
    switch format:
        case "csv":  return CsvExporter(delimiter=",")
        case "tsv":  return CsvExporter(delimiter="\t")
        case "json": return JsonExporter(pretty=true)
        default:     raise Error("Unknown format: " + format)

exporter = exporterFor(format)
```

**Language notes:** Python uses a function or `classmethod`. Java/C# use a static factory method or a dedicated factory class. Rust uses associated functions (`Exporter::for_format`). Go uses a `NewExporter(format) (Exporter, error)` function.

---

## Observer

**Cures:** a module that directly calls every interested party when something changes, hard-coding the list of listeners and coupling unrelated concerns.

```
# BEFORE: the order service knows about email, analytics, inventory...
function placeOrder(order):
    save(order)
    emailService.sendConfirmation(order)     # direct, hard-coded
    analytics.track("order_placed", order)
    inventory.reserve(order)

# AFTER: the service just announces; subscribers react
class OrderEvents:
    listeners = []
    subscribe(fn): this.listeners.add(fn)
    emit(order):   for fn in this.listeners: fn(order)

# wiring (set up once, elsewhere):
orderEvents.subscribe(order -> emailService.sendConfirmation(order))
orderEvents.subscribe(order -> analytics.track("order_placed", order))
orderEvents.subscribe(order -> inventory.reserve(order))

function placeOrder(order):
    save(order)
    orderEvents.emit(order)                  # no knowledge of who listens
```

**Language notes:** many ecosystems have this built in — Node `EventEmitter`, C# `event`/`delegate`, Java `PropertyChangeListener` or a bus library, Rust channels, Kotlin `Flow`. Prefer the native mechanism over a hand-rolled list.

---

## Decorator

**Cures:** the urge to add cross-cutting behavior (logging, caching, retries, auth) by editing a core class or subclassing it many ways.

```
# Core abstraction
interface DataSource: read(key) -> value

class FileDataSource: read(key): return readFromDisk(key)

# Decorators wrap another DataSource and add one behavior each
class CachingDataSource:
    constructor(inner): this.inner = inner; this.cache = {}
    read(key):
        if key in this.cache: return this.cache[key]
        value = this.inner.read(key)
        this.cache[key] = value
        return value

class LoggingDataSource:
    constructor(inner): this.inner = inner
    read(key):
        log("reading " + key)
        return this.inner.read(key)

# Compose behaviors without touching FileDataSource:
source = LoggingDataSource(CachingDataSource(FileDataSource()))
```

**Language notes:** Python often uses function decorators (`@cache`, `@retry`) for the function-level case. In Go, embed the inner interface in a struct. In TS/Java/C#, the wrapping-class form above is idiomatic.

---

## Choosing Between Them

| You have… | Consider |
| --- | --- |
| A switch selecting interchangeable behavior | **Strategy** (or `match`/map of functions) |
| A series of independent checks/handlers | **Chain of Responsibility** (or a list of functions) |
| Scattered, conditional construction logic | **Factory** |
| One change that must notify many reactors | **Observer** |
| Cross-cutting behavior added to existing objects | **Decorator** |

When a functional alternative is shown, prefer it for stateless cases — it's less ceremony and easier to test. Reach for the class-based pattern when you need runtime configuration, shared state, or to satisfy an existing interface.
