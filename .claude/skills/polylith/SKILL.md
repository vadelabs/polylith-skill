---
name: polylith
description: >
  Polylith component architecture for Clojure. Load when working in a Polylith
  workspace (workspace.edn present), creating or modifying components, bases,
  or interfaces, or when asked about Polylith structure and conventions.
user-invocable: true
context:
  - conventions.md
---

# Polylith Architecture

Polylith is a component-based monorepo architecture for Clojure. All code lives in one
repository, organized into isolated bricks that communicate through declared interfaces.

## Workspace Structure

```
workspace/
├── workspace.edn          ← top-namespace and brick registry
├── deps.edn               ← root deps, aliases for each project
├── components/
│   └── <name>/
│       ├── deps.edn
│       ├── src/
│       │   └── <top-ns>/<name>/
│       │       ├── interface.cljc   ← public surface (required)
│       │       └── core.cljc        ← implementation (private)
│       └── test/
│           └── <top-ns>/<name>/
│               └── interface_test.cljc
├── bases/
│   └── <name>/
│       ├── deps.edn
│       └── src/
│           └── <top-ns>/<name>/
│               └── core.cljc
└── projects/
    └── <name>/
        └── deps.edn       ← lists which components + bases to include
```

The top namespace is defined in `workspace.edn` under `:top-namespace`.

## Three Brick Types

**Component** — an isolated unit of business logic with a single responsibility.
- Must have an `interface` namespace (the public surface)
- All other namespaces are private implementation
- Can depend on other components only through their interfaces
- Never depends on a component's implementation namespace directly

**Base** — a thin entry point that wires components together for deployment.
- Has no `interface` namespace (nothing consumes it)
- Consumes component interfaces to build a runnable application
- One base per deployable artifact is typical

**Project** — a deployable artifact configuration.
- Lists which components and bases to include in `deps.edn`
- Does not contain source code itself
- Maps to a build target (uberjar, lambda, etc.)

## Interface Namespace Convention

Every component must have exactly one `interface` namespace:

```
<top-namespace>.<component-name>.interface
```

Example: if `workspace.edn` sets `:top-namespace com.example`, then the `auth` component
exposes `com.example.auth.interface`.

The interface namespace re-exports only what consumers should see:

```clojure
(ns com.example.auth.interface
  (:require [com.example.auth.core :as core]))

(defn authenticate [credentials]
  (core/authenticate credentials))
```

## Dependency Rules

- Components depend only on other component **interfaces**, never implementations
- Bases depend on component interfaces
- Projects declare which bricks to include — they don't contain logic
- Circular dependencies between components are not allowed
- `clojure -M:poly check` enforces all these rules

## Key Insight: Interface Swappability

Multiple components can implement the same interface. This is how Polylith achieves
substitutability without runtime polymorphism:

- `postgres-db` and `sqlite-db` both implement the `db` interface
- Different projects include whichever implementation they need
- Consumers always depend on `<top-ns>.db.interface` — they never change

---

## Supporting References

- Read `conventions.md` when naming functions, namespaces, or reviewing naming choices
- Read `commands.md` when running poly CLI commands or creating/validating bricks
- Read `patterns.md` when creating new components, writing interfaces, or setting up tests
