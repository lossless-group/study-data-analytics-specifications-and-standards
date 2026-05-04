# Data Analytics Specifications and Standards — Study

## The question

> What conventions exist for open, machine-validatable specifications and
> standards across the data-analytics stack — for describing **datasets**,
> **fields**, **dialects**, **columnar storage formats**, **in-memory data
> formats**, and **grammars of graphics** — and what can we learn from the
> mature exemplars before designing our own?

The interesting bits are not the implementations themselves; those are
plentiful. The interesting bits are: how does each spec *encode* the thing
it describes, *validate* conformance, *version* itself, and *compose* with
the layers above and below it in the analytics stack?

## What we are looking at, repo by repo

When reading each entry below, the working checklist is:

- **Spec primitive.** Is the contract expressed as JSON Schema, a Thrift /
  Flatbuffers IDL, RFC 2119 prose, an EBNF grammar, or some mix?
- **Validation surface.** Is there an official validator? Conformance test
  suite? How many independent language drivers exist?
- **Versioning model.** Profile URLs, semver, IDL-level versioning,
  backward-compat policy. How is breakage signaled and managed?
- **Composition.** Where does this spec sit in the stack — wire, file,
  in-memory, logical? What does it reference downward and what references
  it upward?
- **Adoption & governance.** Who maintains it, how do contributions and
  RFCs work, what production systems depend on it?
- **Spec ↔ implementation discipline.** Is the prose authoritative and
  the schema generated, or vice versa? What stops them from drifting?

A mental model of the stack the references collectively cover:

```text
                  ┌───────────────────────────────────────┐
visualization     │  Vega-Lite   Plot   ◄───  ggsql       │
                  │  (JSON spec)  (JS API)                │
                  └─────────────────┬─────────────────────┘
                                    │
                  ┌─────────────────▼─────────────────────┐
data description  │  Data Package                         │
                  │  (FAIR datasets, JSON Schema)         │
                  └─────────────────┬─────────────────────┘
                                    │
                  ┌─────────────────▼─────────────────────┐
in-memory format  │  Apache Arrow                         │
                  │  (columnar, Flatbuffers IDL)          │
                  └─────────────────┬─────────────────────┘
                                    │
                  ┌─────────────────▼─────────────────────┐
on-disk format    │  Apache Parquet                       │
                  │  (columnar, Thrift IDL)               │
                  └───────────────────────────────────────┘
```

Six references, four layers — the visualization layer holds three entries
because the design space *for visualization specs themselves* is the
contested one. Vega-Lite (spec-as-JSON), Plot (spec-as-JS-API), and ggsql
(spec-as-SQL-extension) are three different answers to the same question.

---

## In the study now

### [vega-lite](./vega-lite)
- **Repo:** https://github.com/vega/vega-lite — *A high-level grammar of
  interactive graphics.*
- **Maintainer:** UW Interactive Data Lab (`vega` org); Vega/Voyager
  community.
- **Why this is here:** The canonical visualization grammar that's both
  human-writable and machine-validatable. A Vega-Lite spec is JSON,
  conforms to a published JSON Schema profile, compiles down to lower-level
  Vega, and has been adopted as the rendering target for Altair (Python),
  Streamlit, Observable, Polars/pandas plotting helpers, and more. The
  worked example for "what does a mature visualization spec look like" —
  including how it handles versioning, layered composition, and the messy
  parts (transforms, encodings, projections).

### [plot](./plot) (Observable Plot)
- **Repo:** https://github.com/observablehq/plot — *A concise API for
  exploratory data visualization.*
- **Maintainer:** Observable (`observablehq` org); Mike Bostock and the
  D3 lineage.
- **Why this is here:** The *spec-as-code* counterpoint to Vega-Lite's
  *spec-as-JSON*. Plot is also a grammar of graphics, but its surface is
  a JavaScript API — `Plot.dot(data, {x: "year", y: "gdp"})` — not a JSON
  document. Same underlying grammar (marks, scales, encodings, facets),
  different abstraction. Pinning it lets us study the trade-off head-to-head:
  what does each form give up, what does each gain? Plot's API ergonomics
  for notebook/REPL exploration vs. Vega-Lite's portability and machine
  validatability are the two ends of the design space.

### [datapackage](./datapackage) (Frictionless Data Package)
- **Repo:** https://github.com/frictionlessdata/datapackage — *Data
  Package: a simple container format for describing a coherent collection
  of data.*
- **Maintainer:** Open Knowledge Foundation; Frictionless Data Working
  Group. EU NLnet/NGI0 Entrust funding.
- **Why this is here:** The maturity benchmark. Four interlocking specs
  (Data Package, Data Resource, Table Schema, Table Dialect) where every
  RFC 2119 "MUST" maps onto a JSON-Schema constraint, profiles are
  versioned (`v1` → `v2`, 2024), and 10+ official language drivers exist.
  Aimed squarely at FAIR (Findable, Accessible, Interoperable, Reusable)
  datasets. If the question is "what does a *done* open data spec look
  like?", this is the answer to study.

### [parquet-format](./parquet-format)
- **Repo:** https://github.com/apache/parquet-format — *Apache Parquet
  format specifications.*
- **Maintainer:** Apache Software Foundation; originated at
  Twitter/Cloudera (2013).
- **Why this is here:** The on-disk columnar format spec. Crucially,
  this repo is *not* an implementation — it's the canonical Thrift IDL
  plus prose specification. Multiple independent implementations
  (parquet-cpp, parquet-mr/Java, parquet-rs, arrow's Parquet support) all
  read each other's files because they conform to this spec. The
  reference for "how do you spec a binary file format precisely enough
  for cross-language interop?" — a different problem shape from the
  JSON-schema-style data specs above.

### [arrow](./arrow) (Apache Arrow)
- **Repo:** https://github.com/apache/arrow — *Cross-language development
  platform for in-memory analytics.*
- **Maintainer:** Apache Software Foundation; co-created by Wes McKinney
  (pandas) and Jacques Nadeau (Drill).
- **Why this is here:** The in-memory complement to Parquet. Where Parquet
  specs how columnar data sits *on disk*, Arrow specs how it sits *in
  memory* — defined via Flatbuffers IDL plus prose, with reference
  implementations across 10+ languages (C++, Java, Python, Rust, Go,
  JavaScript, R, Julia, MATLAB, Ruby, …). The modern reference for
  "spec a data format so well that a Rust process and a Python process
  can share memory pages without copying." Worth comparing against
  Parquet's IDL choice (Thrift vs. Flatbuffers) and against Data
  Package's JSON Schema choice — three different answers to "how do
  we spec a data shape."

### [ggsql](./ggsql)
- **Repo:** https://github.com/posit-dev/ggsql — *A SQL extension for
  declarative data visualization based on the Grammar of Graphics.*
- **Maintainer:** Posit (formerly RStudio).
- **Why this is here:** A bridge reference. ggsql extends SQL with
  `VISUALISE`/`DRAW`/`SCALE`/`LABEL` clauses that express a
  ggplot2-style grammar of graphics, and compiles down to **Vega-Lite +
  DuckDB/SQLite**. So in the analytics stack we just drew, ggsql
  composes the query layer with the visualization layer through a single
  spec dialect. Useful for studying how a new spec layer can be designed
  to *inherit* legitimacy from two mature ones (SQL below, Vega-Lite above)
  rather than competing with them. Currently approaching alpha — pinning
  the SHA matters here because the surface is still moving.

---

## Reading order suggestion

1. Start with **Data Package** (`./datapackage`) — read
   `data/specs/data-package/index.md` (or whichever path the spec lives
   at after clone) end-to-end. It's the smallest, most readable, and the
   discipline (RFC 2119 prose ↔ JSON Schema profile) is the cleanest
   example to internalize first.
2. Read **Vega-Lite**'s schema docs (`./vega-lite/site/docs/`) and look at
   `vega-lite-schema.json`. Same JSON-Schema discipline, applied to a
   much larger surface (visualization grammar).
3. Skim **Plot**'s API docs (`./plot/README.md` and the `docs/` folder) as
   a deliberate contrast. Same grammar-of-graphics ideas, but the surface
   is JS function calls, not JSON. Note what becomes ergonomic and what
   becomes harder to validate or transport.
4. Compare against **Parquet** (`./parquet-format`). Read
   `parquet.thrift` first, then the prose docs. Notice how the *primitive*
   changes (Thrift IDL not JSON Schema) but the discipline of "spec is
   authoritative, implementations conform" is the same.
5. Compare against **Arrow** (`./arrow/format/`). Notice the choice of
   Flatbuffers over Thrift, and how the spec lives alongside reference
   implementations rather than as a standalone repo.
6. Finish with **ggsql** as the synthesis case — a new spec defined *by
   composition* with two of the mature specs above (SQL + Vega-Lite).

By the end of that pass, the design space for "what does an open spec
look like in the data-analytics world" should be mostly visible.

## Anti-patterns to watch for in the references

- **Spec drifts behind implementation.** Some projects label their main
  implementation as the "spec" and update it freely; the prose lags. Note
  which references in the list have a discipline against this.
- **Schema-only with no prose.** A JSON Schema or IDL without an RFC-2119
  prose layer leaves intent unstated. Note which references pair them.
- **Single-language reference implementation.** A spec with only one
  implementation isn't really an *open* spec yet — it's a one-vendor API
  with optimistic branding.
