# Checks performed on paper.md

## 1. Spelling

- **Tools used:** `aspell` (en_US, markdown mode), `hunspell` (en_US), plus Claude (Sonnet 5) performing a close read and a regex scan for duplicated/repeated words and common typo patterns (e.g. "teh", "seperate", "it's"/"its" confusion).
- **Result:** no misspellings found. All words flagged by the spell checkers were proper nouns, citation keys, LaTeX macros, or valid technical/compound terms (e.g. `queryable`, `lifecycle`, `oversized`, `parametrizing`, `petabases`).
- **Output:** none (remarks.md was not populated for this pass — nothing to report).

## 2. Grammar and unclear sentences

- **Tools used:** `proselint` (installed via `pip install --user proselint`), plus Claude (Sonnet 5) performing a sentence-by-sentence close read for grammar errors, ambiguous referents, and unclear phrasing that `proselint` cannot judge (style linters catch surface patterns like passive voice or wordiness, not whether a sentence is genuinely unclear).
- **Result:** 11 remarks written to `remarks.md`, covering parallelism issues, ambiguous referents, awkward word order, and missing articles.
- **Additional issues surfaced by `proselint` but outside the grammar/clarity scope (reported in chat, not added to remarks.md):**
  - l88: leftover `<!-- TODO before submission -->` comment in the source.
  - l97, l114: straight quotes instead of curly quotes (typographic, not grammar).

## 3. Citations (bibliography vs. in-text)

- **Tools used:** `grep`/`comm` cross-check of `@key` citations in `paper.md` against entries in `paper.bib` (missing, unused, and duplicate keys), plus inspection of the CI-built `paper.pdf` (via `pdftotext -layout`) to confirm every citation actually renders, performed by Claude (Sonnet 5).
- **Result:** all 17 citation keys used in `paper.md` have a matching `paper.bib` entry; no unused or duplicate bib entries; all 17 references render cleanly in the compiled PDF (no broken `??` refs or leftover `[@key]` markup).
- **Findings:**
  - `karasikov2025`, `sra`, and `blaxter2025` are missing a `doi` field, unlike every other entry in `paper.bib` — worth adding for consistency and so readers get a persistent link.
  - The bib key `harris2019` implies year 2019, but the entry's actual `year` field is 2020 (matches its real publication date), so the rendered citation correctly reads "(Harris & Medvedev, 2020)". Purely a cosmetic key/year mismatch in `paper.bib`, doesn't affect the printed paper, but could confuse whoever edits the bib file next.

## 4. Figures and tables

- **Tools used:** `grep` for figure/table references in `paper.md`, `ls`/`git status` on `figures/`, and inspection of the compiled PDF for figure numbering, performed by Claude (Sonnet 5).
- **Result:** one figure in the document (`figures/workflow_V2.pdf`, referenced at l86 and cross-referenced as "Figure 1" at l72); numbering resolves correctly in the compiled PDF ("Figure 1: Overview of the kmhelpers workflow"). No tables in the document.
- **Findings:**
  - `figures/pipeline_metro.drawio` is currently modified in the working tree but is not referenced anywhere in `paper.md` — confirm whether it's a work-in-progress replacement/addition still to be exported and linked, or unrelated in-progress work.
  - l88: the TODO comment flagging that the figure still needs to show index update is unresolved (added to `remarks.md`).

## 5. LaTeX / math rendering

- **Tools used:** `grep` to check `$...$` math-span balance (40 `$` characters = 20 balanced pairs) in `paper.md`, manual inspection of each math expression, and comparison against the compiled `paper.pdf` (`pdftotext -layout`) to see how each one actually rendered, performed by Claude (Sonnet 5). `pandoc`/`xelatex` are not installed locally, so a full local recompile wasn't possible — JOSS builds via the `openjournals/openjournals-draft-action` Docker image in CI (`.github/workflows/draft-pdf.yml`).
- **Result:** all math spans are well-formed and render correctly in the existing CI-built PDF (e.g. 𝒮 = {𝑆1, …, 𝑆𝑛}, ≥ 3.8, 𝑘-mer). No unescaped LaTeX special characters or stray backslashes outside math mode.
- **Caveat:** the comparison PDF was built from the last committed `paper.md`; the only uncommitted diff since then is one added ORCID and a wording tweak ("as illustrated in Figure 1"), neither touching math or citations, so the comparison remains valid for this check.

## 6. JOSS-specific metadata/schema validation

- **Tools used:** the `inara`/`openjournals-draft-action` tooling itself could not be run locally — no Docker daemon available, and `inara` is not a published RubyGem (`gem install inara` fails; it's only distributed as part of the Docker action). Instead, Claude (Sonnet 5) fetched JOSS's own authoring spec (`docs/paper.md` from `openjournals/joss` on GitHub, the source for https://joss.readthedocs.io/en/latest/paper.html) and checked the YAML frontmatter of `paper.md` field-by-field against it, then cross-checked the result against the CI-built `paper.pdf`.
- **Result — CONFIRMED BUG:** the `date` field (l26) is `2026-07-06` (ISO format), but JOSS requires the exact format `%e %B %Y` (e.g. `9 October 2024`). This isn't cosmetic: the compiled `paper.pdf` shows **"Submitted: 01 January 1970"** in the sidebar — the Unix epoch — proving the date parser silently failed on the ISO string instead of using the intended date. Fix: change l26 to `date: 6 July 2026`.
- **Other frontmatter checked, all OK:**
  - `title`, `tags`, `bibliography: paper.bib` — present and correctly formatted.
  - `authors`: each has `name`, `orcid`, and `affiliation`; all three ORCIDs are syntactically valid (4 groups of 4 alphanumerics, last character may be `X`).
  - `affiliations`: each has `index` and `name`; author `affiliation` values (1, 2, 1) all resolve to a defined index — no dangling references.
  - Figure syntax (l86) is a standalone paragraph (blank line before/after) as JOSS requires for it to be treated as a captioned figure, not an inline image.
  - Body word count (Summary through Acknowledgements) is 1,666 words, within JOSS's required 750–1750 range.
- **Minor/unconfirmed:** section headings differ in capitalization from JOSS's canonical spelling — l40 "Statement of Need" vs. canonical "Statement of need", l53 "State of the Field" vs. "State of the field", l116 "AI Usage Disclosure" vs. "AI usage disclosure". JOSS's own docs use lowercase for the second word in these headings; unclear whether automated validation is case-sensitive (this appears to be a manual editor-checklist item, not a machine-enforced schema rule), but aligning casing with the canonical template costs nothing and removes any doubt.

## Not yet checked

- Full local recompilation of `paper.md` → PDF (no `pandoc`/`xelatex` locally; relies on CI-built `paper.pdf` as reference). This is also why the `date` bug above wasn't testable by re-running the build after a fix — recommend pushing the fix and letting the CI action rebuild to confirm "Submitted:" now shows the correct date.
