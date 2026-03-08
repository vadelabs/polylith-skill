---
description: Common Polylith implementation patterns for interfaces, cross-platform code, dependencies, and tests
---

# Common Polylith Patterns

## The Interface Pattern

Every component exposes exactly one public namespace: `interface`. Everything else is
private. Consumers import the interface and nothing else.

```clojure
;; src/com/example/auth/interface.cljc
(ns com.example.auth.interface
  (:require [com.example.auth.core :as core]))

;; Re-export only what consumers need
(defn authenticate [credentials]
  (core/authenticate credentials))

(defn valid-token? [token]
  (core/valid-token? token))
```

The implementation lives in `core.cljc` (or any private namespace):

```clojure
;; src/com/example/auth/core.cljc
(ns com.example.auth.core
  (:require [com.example.db.interface :as db]))  ; ← only interface!

(defn authenticate [credentials]
  (when-let [user (db/find-user (:email credentials))]
    (when (valid-password? (:password credentials) (:hashed-password user))
      user)))
```

## Multiple Implementations of One Interface

Define the interface structure once. Let multiple components implement it.

```
components/
├── postgres-db/
│   └── src/com/example/db/interface.cljc   ← PostgreSQL implementation
├── sqlite-db/
│   └── src/com/example/db/interface.cljc   ← SQLite implementation
└── mem-db/
    └── src/com/example/db/interface.cljc   ← in-memory test implementation
```

Each implements the same namespace path, so consumers never change:

```clojure
;; This import works regardless of which db component is in the project
(require '[com.example.db.interface :as db])
(db/find-user email)
```

Projects select which implementation to include in their `deps.edn`.

## Cross-Platform .cljc Files

Use reader conditionals for platform-specific code:

```clojure
(ns com.example.time.core)

(defn now []
  #?(:clj  (java.time.Instant/now)
     :cljs (js/Date.)))

(defn epoch-ms []
  #?(:clj  (.toEpochMilli (java.time.Instant/now))
     :cljs (.getTime (js/Date.))))
```

Reader conditionals are only needed when the imports actually differ between platforms:

```clojure
(ns com.example.my-component.core
  (:require
    [clojure.string :as str]                        ; same on both — no conditional needed
    [com.example.other.interface :as other]
    #?(:clj  [clojure.java.io :as io]               ; platform-specific — needs conditional
       :cljs [cljs.reader :as reader])))
```

Platform-only files use `.clj` or `.cljs` extensions instead of `.cljc`.

## Adding a Component Dependency

1. Add the dependency to the component's `deps.edn`:

```clojure
;; components/auth/deps.edn
{:deps {com.example/db {:local/root "../../components/db"}}}
```

2. Require the interface (not the implementation) in your code:

```clojure
(require '[com.example.db.interface :as db])
```

3. Add the dependency to any project that uses `auth`:

```clojure
;; projects/api/deps.edn
{:deps {com.example/auth {:local/root "../../components/auth"}
        com.example/db   {:local/root "../../components/db"}}}
```

4. Run `clojure -M:poly check` to validate the dependency graph.

## Test Structure

Tests live in the component alongside source. Test the interface, not internals:

```clojure
;; test/com/example/auth/interface_test.cljc
(ns com.example.auth.interface-test
  (:require
    #?(:clj  [clojure.test :refer [deftest is testing]]
       :cljs [cljs.test    :refer [deftest is testing]])
    [com.example.auth.interface :as auth]))

(deftest authenticate-test
  (testing "valid credentials return a user"
    (is (some? (auth/authenticate {:email "user@example.com"
                                   :password "correct"}))))

  (testing "invalid credentials return nil"
    (is (nil? (auth/authenticate {:email "user@example.com"
                                  :password "wrong"})))))
```

## CLAUDE.md for Polylith Projects

A `CLAUDE.md` at the workspace root with your component list and test command:

```markdown
# Project Name

Polylith monorepo. Top namespace: `com.example`.

Load the polylith skill for architecture rules and conventions.

## Components
- `auth` — authentication and session management
- `db` — database access (postgres implementation)
- `email` — transactional email via SendGrid

## Running tests
clojure -M:poly test
```

List your components so Claude can navigate without guessing.
