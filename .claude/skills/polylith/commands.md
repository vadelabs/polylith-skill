---
description: Polylith poly CLI command reference for workspace inspection, validation, and brick creation
---

# Polylith CLI Commands

The `poly` tool is invoked via a Clojure alias (typically `:poly`).

> **Note**: Many projects wrap poly commands in Babashka tasks (e.g., `bb check`,
> `bb info`). Check the project's `bb.edn` for available aliases.

## Workspace Inspection

```bash
# Summarize workspace: bricks, projects, changed/stable state
clojure -M:poly info

# Full workspace data as EDN
clojure -M:poly ws

# Show library dependencies
clojure -M:poly libs

# Show what changed since last stable point (git tag)
clojure -M:poly diff
```

## Validation

```bash
# Validate workspace structure and dependency rules
clojure -M:poly check
```

Run `check` after any structural change. It catches:
- Components depending on other components' implementation namespaces
- Missing interface namespaces
- Circular dependencies
- Misconfigured projects

## Creating Bricks

```bash
# Create a new component
clojure -M:poly create component name:my-component

# Create a new base
clojure -M:poly create base name:my-base

# Create a new project
clojure -M:poly create project name:my-project
```

After creating a component:
1. Implement `src/<top-ns>/my-component/interface.cljc` — the public surface
2. Add implementation in `src/<top-ns>/my-component/core.cljc`
3. Write tests in `test/<top-ns>/my-component/interface_test.cljc`
4. Add the component to relevant project `deps.edn` files

## Running Tests

```bash
# Run tests for changed bricks only
clojure -M:poly test

# Run tests for a specific project
clojure -M:poly test project:my-project

# Run all tests regardless of change detection
clojure -M:poly test :all
```

## Aliases

Most projects define project-specific test aliases in the root `deps.edn`:

```bash
# Run dev environment tests (check your project's deps.edn for aliases)
clojure -M:dev:test
```

## Stable Points

Polylith uses git tags to track "stable" commits. Tests only run for bricks
that changed since the last stable point.

```bash
# Mark current HEAD as stable
git tag -f stable-$(git rev-parse --short HEAD)
```
