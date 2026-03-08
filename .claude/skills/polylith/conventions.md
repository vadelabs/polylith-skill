---
description: Clojure naming conventions for functions, predicates, constructors, and accessors
---

# Clojure Naming Conventions

Names describe *what*, not *how*.

## Pure Functions — Use Nouns

Pure functions return values. Name them for what they return.

```clojure
;; Good
(defn age [birthdate] ...)
(defn price [product] ...)
(defn normalized-path [path] ...)

;; Bad — unnecessary verb prefix
(defn calculate-age [birthdate] ...)
(defn get-price [product] ...)
```

## Side-Effecting Functions — Use Verbs

Functions that perform I/O, write to state, or make network calls.

```clojure
(defn send-email [to subject body] ...)   ; I/O
(defn save-user [user] ...)               ; database write
(defn fetch-config [url] ...)             ; HTTP request
(defn load-config [path] ...)             ; file read
```

## Predicates — End with `?`

```clojure
(defn valid? [x] ...)
(defn authenticated? [session] ...)
(defn in-stock? [product] ...)
```

## Don't Repeat the Namespace

Function names should not restate the namespace they live in.

```clojure
(ns products)

;; Bad
(defn product-price [p] ...)
(defn product-name [p] ...)

;; Good
(defn price [p] ...)
(defn name [p] ...)

;; Usage: (products/price p)
```

## Constructors and Builders

For pure construction (no side effects), name for the output type:

```clojure
(defn config [opts] ...)       ; returns a config map
(defn resource [uri name] ...) ; returns a resource map
```

For constructors with side effects:

```clojure
(defn create-user [attrs] ...)       ; writes to database
(defn create-connection [url] ...)   ; opens network connection
```

## Coercions vs Conversions

Coercions accept multiple input types and produce a target type:

```clojure
(defn file [x] ...)     ; anything → File
(defn uri [x] ...)      ; anything → URI
```

Conversions name both ends when multiple conversion functions exist:

```clojure
(defn string->int [s] ...)
(defn map->json [m] ...)
```

## Accessors — No `get-` Prefix for Pure Access

```clojure
;; Good
(defn config [] @config-atom)
(defn working-dir [config] (:working-dir config))

;; Bad
(defn get-config [] ...)
(defn get-working-dir [config] ...)
```

Use `fetch-` or `load-` when the accessor has side effects.

## Exclamation Mark `!`

Only for functions that mutate a reference (atom, ref, agent):

```clojure
(defn reset-state! [] (reset! state-atom {}))

;; Do not use ! for I/O — that's what verbs are for
(defn save-file [path content] ...)   ; no !
```

## Interface Namespace Names

In Polylith, the interface namespace is the component's public surface.
Name functions in it as you would any other namespace — the `interface` part
of the namespace path is invisible to callers who alias it.

```clojure
;; Component: auth
;; Namespace: com.example.auth.interface

(defn authenticate [credentials] ...)   ; not auth-authenticate
(defn valid-token? [token] ...)
(defn session [user] ...)               ; pure — returns a session map
(defn create-session [user] ...)        ; writes to store — verb
```
