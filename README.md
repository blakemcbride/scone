# Scone Knowledge-Base System

> **About this fork.** This repository is a fork of Scott Fahlman's
> Scone system, maintained by Blake McBride
> ([https://github.com/blakemcbride](https://github.com/blakemcbride)).
> The upstream engine and knowledge bases are unchanged in substance;
> the fork adds quality-of-life and documentation improvements listed
> in [Changes in this Fork](#changes-in-this-fork) below.

Scone is a knowledge representation and reasoning system — a knowledge-base
system (KBS) — developed by Scott Fahlman's research group in the Language
Technologies Institute of Carnegie Mellon University. Scone, by itself, is
not a complete AI or decision-making system, and does not aspire to be;
rather, it is a software component — a sort of smart active memory system —
designed to be used in a wide range of software applications, both in AI
and in other areas. Scone deals just with symbolic knowledge. Things like
visualization, motor memory, and memory for sound sequences are also
important for human-like intelligence, but we believe that those will have
specialized representations of their own, linked in various ways to the
symbolic memory.

Scone occupies a different part of the design space from other knowledge-base
systems currently in use — particularly systems such as OWL that are based
on First-Order Logic or Description Logic. Our goal in developing Scone has
been to support common-sense reasoning and natural-language understanding,
not theorem-proving and logic puzzles. Therefore, we place primary emphasis
on Scone's expressiveness, ease of use, and scalability.

In addition to making Scone available to a larger community, and building
up the necessary supporting machinery, our goals include adding more
episodic representation and reasoning capabilities to Scone. By "episodic"
we mean the ability to represent and reason about events, actions,
sequences, plans, goals, time durations, processes and rates, and so on.
In addition, we are working on knowledge-based natural language
understanding based on Scone.

## What's in This Repository

| Path | Description |
| --- | --- |
| `engine.lisp` | The Scone engine — a single large Common Lisp file. |
| `kb/` | Core knowledge-base files (bootstrap, common-sense core ontology, optional WordNet lexical components). |
| `scone-loader.lisp` | Convenience loader that starts the engine and loads the bootstrap KB. |
| `manual/` | The Scone User's Guide and step-by-step Tutorial (LaTeX source, plus pre-built PDFs). |
| `Scone-Users-Guide.docx` | Original Word version of the User's Guide. |
| `Release-Notes.txt` | User-visible changes between releases. |
| `Apache-License-2.0.txt` | License text. |

## Requirements

Scone is written in Common Lisp. It is primarily developed and tested with
[Steel Bank Common Lisp (SBCL)](http://www.sbcl.org/), and has also been
run successfully on CMUCL, CLISP, LispWorks, and Allegro Common Lisp.

## Building

There is no build step. The engine is loaded as source into a running
Common Lisp image.

## Running

The recommended way to start Scone is via `scone-loader.lisp`. Start
your Lisp from the top level of this repository and evaluate:

```lisp
(load "scone-loader.lisp")
(scone)
(load-kb "core")
```

- `(load "scone-loader.lisp")` reads the loader and prints a reminder
  of how to invoke it.
- `(scone)` loads `engine.lisp`, sets `*default-kb-pathname*` to point
  at this repository's `kb/` directory, and loads the bootstrap KB
  (`{thing}`, `{set}`, and the other structures the engine itself
  refers to).
- `(load-kb "core")` pulls in the generic common-sense core ontology
  (upper, units, space, time, simple physics, biology, information,
  social, person, episodic, and geopolitics).

Additional KB files can then be loaded as needed, for example the
(large) WordNet lexical component:

```lisp
(load-kb "lexical-components/wordnet-concepts")
```

### Alternative: loading `engine.lisp` directly

If you prefer not to use the loader, you can perform the same steps by
hand:

```lisp
(load "engine.lisp")

(setf *default-kb-pathname*
      (merge-pathnames "kb/anonymous.lisp" (truename ".")))

(load-kb "bootstrap")
(load-kb "core")
```

`*default-kb-pathname*` is used as the default by `load-kb`; set it once
to any file inside your `kb/` directory (the filename part is overridden
per call) and subsequent `load-kb` calls resolve relative to it.

See the Scone User's Guide in the `manual/` directory for a full tour of
the API, and the companion `manual/Scone-tutorial.pdf` for a hands-on,
step-by-step introduction that builds a small zoo/animals ontology from
scratch.

## Status and Support

The core capabilities of Scone have been running for several years, and
have been used in a number of projects at CMU and by a few outside
collaborators. It has always been our intention to release Scone as an
open-source resource for the research community and other potential users.
Our research group is very small at present, and we are all very busy on
our ongoing research efforts on Scone and related applications (plus the
never-ending quest to keep the project funded). So we welcome comments,
queries, contributions, suggestions, and even requests for new
functionality, but we cannot promise anything in the way of timely support
for external users and potential contributors. So for now, Scone is for
the adventurous and for those who can answer some of their own questions
by looking at our code and assorted documents and papers.

## More Information

- [Scone project home page](http://www.cs.cmu.edu/~sef/scone/)
- [Scott Fahlman's Knowledge Nuggets blog](http://www.cs.cmu.edu/~nuggets)
- Scott E. Fahlman, *NETL: A System for Representing and Using
  Real-World Knowledge*, MIT Press, 1979 — the original book-length
  treatment of the marker-passing approach that Scone continues and
  extends.

## Changes in this Fork

This fork, maintained by
[Blake McBride](https://github.com/blakemcbride), layers the following
changes on top of the upstream CMU release. The engine semantics and
the shipped knowledge bases are unchanged.

- **`scone-loader.lisp` is now portable.** It locates `engine.lisp`
  and `kb/` relative to the loader file itself, so `(scone)` works
  immediately after cloning. The upstream version required users to
  hand-edit hardcoded `/Users/sef/...` paths before first use.
- **`engine.lisp` loads cleanly under SBCL.** Added a block of forward
  `(declaim (ftype function ...))` declarations for the engine's
  mutually-recursive functions, silencing ~60 `STYLE-WARNING: undefined
  function` messages that SBCL (and other strict compilers) emitted on
  first load. No runtime behavior changed.
- **Rewritten `README.md`** with a repository map, requirements,
  explicit run instructions, and both the recommended
  (`scone-loader.lisp`) and alternative (`engine.lisp` direct) startup
  paths.
- **Added `CLAUDE.md`** — a short orientation document covering the
  engine-vs-KB split, the mandatory bootstrap/core load order, and the
  deferred-connections mechanism, intended to get a new contributor
  productive quickly.
- **Rebuilt the Scone User's Guide as LaTeX.**
  `manual/Scone-Users-Guide.tex` is a full conversion of the original
  Word document with:
  - two-sided layout suitable for printing on 8.5 × 11 paper, with
    asymmetric binding-side margins;
  - each top-level section starting on its own right-hand (recto) page,
    book-style, via `titlesec` + `\cleardoublepage`;
  - a live `\tableofcontents` (replacing a stale hand-copied TOC);
  - colored cross-reference links;
  - a new **Getting Started** chapter covering startup, KB loading,
    checkpointing, and exiting;
  - a **Dependencies between KBs** note explaining that the shipped
    KBs are additive and layered, not mutually exclusive;
  - a pointer to Fahlman's 1979 *NETL* book in the Overview;
  - substantially expanded API coverage to match what is actually
    exported from `engine.lisp`: wires (`connect-wire`,
    `disconnect-wire`, `disconnect-split-wires`), low-level marker
    management (`lock-marker`, `unlock-marker`, `locked-marker?`,
    `available-marker?`, `context-marker-setup`) and bit predicates
    (`all-bits-on?`, `any-bits-on?`, `all-bits-off?`), namespace and
    iname internals (`make-namespace`, `get-namespace`,
    `extract-namespace`, `lookup-element-iname`, `element-name`,
    `iname`, `print-element-iname`, `read-element`), deferred-connection
    APIs (`defer-connection`), `dump-element`, a **Cardinality** note
    (documenting that `does-x-have-a-y?` and friends are currently
    stubbed out in the engine), and a new top-level **Consistency
    Checking** section covering `find-split-violations`,
    `violated-split?`, `split-violations-below?`, `incompatible?`, and
    `find-path`.
  A pre-built PDF (`manual/Scone-Users-Guide.pdf`) is included; the
  original `.docx` is preserved at the repository root.
- **Added a step-by-step tutorial.**
  `manual/Scone-tutorial.tex` (with pre-built `Scone-tutorial.pdf`) is a
  new, hands-on companion to the User's Guide. It uses the same
  two-sided, book-style LaTeX layout and walks a new user through
  starting Scone, creating types and individuals, building an is-a
  hierarchy, listing and showing elements, defining roles and
  statements, enforcing disjointness with splits, overriding inherited
  properties with cancellation, using contexts for hypothetical worlds,
  and checkpointing a KB. Each chapter ends with a short exercise. The
  running example is a small zoo / animals ontology.

## License

Scone is released under the Apache License 2.0. See
`Apache-License-2.0.txt` for the full license text.
