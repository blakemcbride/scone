# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Scone is

Scone is a symbolic knowledge-representation and reasoning system written in Common Lisp, developed by Scott Fahlman's group at CMU. It is a "smart active memory" component intended to be embedded in larger AI applications — not a complete reasoner or theorem-prover. The emphasis is on expressiveness, common-sense reasoning, and scalability over logical completeness. Authoritative user-facing documentation is `Scone-Users-Guide.docx` (SUG); release-level notes are in `Release-Notes.txt`.

## Running Scone

There is no build system and no automated test suite — Scone is loaded interactively into a Common Lisp image (typically SBCL, CCL, or Allegro).

1. **Fix the hardcoded paths in `scone-loader.lisp` before first use.** The file ships with `/Users/sef/scone/~A/...` pathnames pointing at the original author's filesystem. Update both the `*default-kb-pathname*` format string and the `(load ...)` engine pathname to point at this repository, or rely on `*default-kb-pathname*` being rebound by your own loader.
2. Start a Lisp, `(load "scone-loader.lisp")`, then `(scone "some-version-name")`. That loads `engine.lisp`, enters the `:scone` package if one was created, and loads `kb/bootstrap.lisp` via `LOAD-KB`.
3. From then on, load additional KB files with `(load-kb "core")` (the common-sense core) or individual components like `(load-kb "core-components/time-model")`. `LOAD-KB` is the only supported way to ingest KB files — it does bookkeeping around `*loaded-files*`, deferred connections, and element IDs that bare `LOAD` does not.

Persist a modified KB with `(checkpoint-kb "filename")`; the result is a text `.lisp` file re-loadable with `LOAD-KB` on top of `bootstrap`.

## Architecture

**Engine vs. KB is the fundamental split.** `engine.lisp` is a single ~6800-line file containing every primitive: element structures, marker bits, is-a/eq/has/cancel links, splits, relations, statements, contexts, the marker-scan algorithms, the namespace/dictionary machinery, and the KB loader. Everything else in the repo is *data* expressed as Lisp forms that call into engine APIs.

The engine is marker-based: reasoning happens by propagating bits through a network of elements. A compiler declaim at the top of `engine.lisp` pins scan code to `(optimize (speed 3) (space 0) (safety 0))`, and the style comment in the header explicitly says marker ops are optimized for speed over clarity while KB-building code is optimized for clarity. Respect that split when editing — do not slow down scan paths with safety/consing, and do not over-optimize the construction paths. `engine.lisp` has a "TABLE OF CONTENTS" comment block (~line 54) listing every section; use it to navigate.

**KB layering is load-order sensitive.** The dependency chain is:

- `kb/bootstrap.lisp` — **must load before anything else and before any engine function that builds elements**. Defines `{thing}`, `{set}`, and other elements that engine code refers to through global variables like `*thing*`. Sets `*defer-unknown-connections* t` because bootstrap inherently contains forward references.
- `kb/core.lisp` — orchestrator that `LOAD-KB`s the generic common-sense ontology in the order: upper → units → space-model → time-model → simple-physics → simple-biology → simple-information → simple-social-model → simple-person-model → simple-episodic-model → simple-geopolitics. Later files depend on earlier ones; do not reorder casually.
- `kb/core-components/` — the individual files listed above.
- `kb/lexical-components/` — very large WordNet-derived KBs (`wordnet-concepts.lisp`, `wordnet-english-names.lisp`, `wordnet-spanish-names.lisp`, each multi-megabyte). Load these only when you actually need lexical coverage; they are slow and not required for core reasoning.

**Element naming.** The `{curly-brace}` reader syntax in KB files looks up elements by external name in the current namespace/context. `in-namespace` and `in-context` calls at the top of each KB file set the scope. If you see a bare `{foo}` in a KB file, its binding depends on the surrounding `in-namespace`/`in-context`.

**Deferred connections.** When a KB file references an element not yet defined, the reference is pushed onto `*deferred-connections*` if `*defer-unknown-connections*` is set, and resolved at end-of-file by `process-deferred-connections`. Any refactor touching load ordering or element creation must preserve this dance — the bootstrap file specifically relies on it.

## Working on this codebase

- Changes to `engine.lisp` are almost always reflected in `Scone-Users-Guide.docx`. The SUG and Release-Notes.txt document user-visible API changes (e.g., the `LOOKUP-ELEMENT` / `LOOKUP-ELEMENT-OR-DEFER` rename described in `Release-Notes.txt`). If you rename or change the contract of an exported function, flag that the SUG will need a matching update — do not silently edit the `.docx`.
- Several globals control load-time verbosity and checking: `*verbose-loading*`, `*no-kb-error-checking*`, `*comment-on-element-creation*` (default T), `*comment-on-defined-types*` (default NIL), `*check-defined-types*`, `*create-undefined-elements*`, `*deduce-owner-from-type-role*`. Toggle these rather than editing engine code when diagnosing load issues.
- Sanity check after engine changes by loading `scone-loader.lisp` → `(scone ...)` → `(load-kb "core")` → `(load-kb "lexical-components/wordnet-concepts")` in a fresh Lisp and watching for errors. That is the de-facto smoke test.
