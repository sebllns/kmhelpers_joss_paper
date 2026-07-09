## Front matter

l26: the `date` field `2026-07-06` (ISO format) does not match JOSS's required `%e %B %Y` format. This is a confirmed bug, not a style preference: the compiled paper.pdf shows "Submitted: 01 January 1970" in the sidebar because the date parser silently fails on this format. Change to `date: 6 July 2026`.

## Summary

l33: "Bloom filters (BF)" defines the plural noun with a singular abbreviation. Later the text uses the plural form "BFs" (e.g. l65, l68), so the abbreviation should be introduced as "(BFs)" here to match.

l34: this sentence is very long (sizing, configuring, distributing, bounding, grouping — five parallel clauses in one sentence). Consider splitting into two sentences for readability.

## Statement of Need

find:"answer queries on large-scale samples":unclear phrasing — "large-scale samples" reads as if individual samples are large-scale, but the intended meaning is probably that queries operate at large scale across many samples. Consider rewording, e.g. "answer queries at scale across large sample collections".

find:"requires a user to master, simultaneously:":awkward placement of "simultaneously" right before the colon. Consider "requires a user to simultaneously master:" for smoother flow.

find:"This expertise is not required to obtain":the elliptical construction ("not required to obtain X but to obtain Y") is easy to misparse on first read. Consider making it explicit: "This expertise is not needed to obtain a functional index, but it is needed to obtain an efficient one."

find:"This opens to non-specialists both the creation and maintenance":awkward word order. Consider "This opens both the creation and maintenance of optimized indexes to non-specialists."

find:"either a raw sequencing dataset or assembled sequences":parallelism issue — "a raw sequencing dataset" (singular) vs. "assembled sequences" (plural). Consider "either a raw sequencing dataset or a set of assembled sequences."

## State of the Field

(no remarks)

## Software design

find:"wastes enormous storage.":slightly odd collocation — "storage" alone reads incomplete here. Consider "wastes enormous amounts of storage" or "wastes enormous storage space".

find:"while remaining flexible enough for large-scale production use.":unclear referent — grammatically "remaining flexible" could attach to the immediately preceding "workflows" or to the earlier compound subject ("the declarative YAML index-definition format and the plan/apply separation"), but the intended meaning is presumably the latter (the design stays flexible). Consider rephrasing to make the subject explicit, e.g. "...while the format and the plan/apply separation remain flexible enough for large-scale production use."

## Research impact statement

find:"Tree of Life Programme at the Wellcome Sanger Institute is one of the major contributors":missing definite article — should read "The Tree of Life Programme at the Wellcome Sanger Institute is one of the major contributors". The same omission recurs later ("To date, Tree of Life has already released...") — check whether "Tree of Life" is meant to be used consistently as a bare proper noun or should take "The" throughout.

find:"sequencing hybrid specimens, or mistakes in sample handling":unclear/parallelism — in the list "cryptic species, sequencing hybrid specimens, or mistakes in sample handling", the middle item is a gerund phrase (an action) while the other two are noun phrases (a condition and an event). Clarify whether the intended meaning is "sequencing of hybrid specimens" (i.e. a hybrid individual was sequenced) and consider rewording for parallel structure.

find:"This project is a perfect use case for applying `kmhelpers`: from input data":this sentence is long and holds two em-dash asides plus a colon-introduced clause. Consider splitting into two sentences for clarity.

## The pipeline

(no remarks)

## Acknowledgements

find:"and by the state funding managed by the French National Research Agency":the definite article reads oddly here since "state funding" has not been previously introduced as a specific, known entity. Consider "and by state funding managed by the French National Research Agency..." (drop "the").
