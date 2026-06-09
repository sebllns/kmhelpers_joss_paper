---
title: 'kmhelpers: A Python toolkit for automated management of genomic indexes'
tags:
  - Python
  - bioinformatics
  - k-mer indexing
  - kmindex
  - genomics
  - sequence querying
  - Bloom filters
authors:
  - name: Sébastien Bellenous
    affiliation: 1
  - name: Pierre Peterlongo
    orcid: 0000-0003-0776-6407
    affiliation: 1
affiliations:
  - name: Genscale, Univ. Rennes, Inria, CNRS, IRISA - UMR 6074, Rennes, F-35000 France
    index: 1
date: 2026-06-03
bibliography: paper.bib
---


# Summary

Large-scale genomic sequence search has become a central problem in modern
bioinformatics [@marchet2024; @karasikov2024]. A widely used family of methods
relies on Bloom filters (BF) [@bloom] to record, for each indexed sample, which
$k$-mers (words of length $k$) it contains; a query then reports the set of
samples containing each queried $k$-mer. Building such an index over many samples
is not a single operation but a chain of interdependent steps, and each step
demands specialist knowledge: sizing each Bloom filter to its sample, configuring
the third-party components used internally by `kmindex` (such as the `fimpera`
approximate-membership layer), distributing data and computation, bounding peak
RAM, and grouping samples into sub-indexes so that the final index stays small
without slowing queries. An error at any step can invalidate downstream results,
waste significant computation, or produce an oversized index.

We present `kmhelpers`, an open-source Python toolkit that automates this entire
workflow for indexes built with `kmindex` [@lemane2024]. It hides the technical
decisions listed above behind a command-line interface (CLI) and a matching
Python API that cover every stage of the $k$-mer index lifecycle, from raw
samples to queries, including the federation of independently built indexes into
a single queryable resource.


# Statement of Need

$k$-mer-based sequence databases built with tools such as `kmindex` [@lemane2024]
and `kmtricks` [@lemane2022] answer large-scale questions in genomics,
metagenomics, and population studies; they were recently used to index 50
petabases of raw sequencing data [@ls]. Yet a wide gap separates having the
indexing software installed from running a reproducible, end-to-end,
size-optimised indexing workflow.

Crossing that gap currently requires a user to master, simultaneously: Bloom
filter sizing as a function of each sample's $k$-mer cardinality; the
configuration of `kmindex`'s internal components; the distribution of data and
build jobs across the available compute; the control of peak memory; and the
partitioning of samples into sub-indexes that balances index size against query
speed. Only specialists hold all of this knowledge at once.

`kmhelpers` removes that barrier. It makes the complete process of building,
updating, and querying a `kmindex` search engine accessible to any group that
holds a large, evolving genomic dataset — research teams, sequencing platforms,
hospitals, or data-production centres — whether the resulting index is kept
private or shared. This opens to non-specialists both the creation and
maintenance of private indexes and the contribution of sub-indexes to larger,
federated search engines.

Formally, the input is a set $\mathcal{S}=\{S_1, \dots, S_n\}$ of $n$ genomic
samples of various sizes; each $S_i$ is either a raw sequencing dataset or
assembled sequences. The output is a `kmindex` index subject to two user-defined
limits: (1) the maximum allowed false-positive rate, and (2) the maximum number
of sub-indexes, each a file storing a set of BFs as a matrix. `kmhelpers`
orchestrates every step between input and output, automating the parametrisation
and the data distribution.


# State of the Field

Several tools tackle large-scale sequence search with $k$-mer-based data
structures. BIGSI [@bradley2019] and COBS [@bingmann2019] use compressed Bloom
filter matrices to answer presence/absence queries across collections of
sequencing experiments. HowDeSBT [@harris2019] and Mantis [@pandey2018] rely on
sequence Bloom trees and counting quotient filters, respectively. More recent
colored-$k$-mer indexes such as MetaGraph [@karasikov2024] and Fulgor [@fulgor]
push scalability further; see [@marchet2024] for a survey. `kmindex`, which
`kmhelpers` wraps, builds on the `kmtricks` [@lemane2022] counting pipeline and
is designed for efficient querying of large sequence collections.

`kmhelpers` does not compete with any of these approaches at the algorithmic
level. Its contribution is orthogonal and, importantly, is not merely a pipeline
script: the difficulty here lies not in chaining steps (any workflow manager or
scripting language can do that) but in the individual components and how they are
orchestrated. `kmhelpers` implements the decisions a `kmindex` expert would
otherwise make by hand: `profile` computes Bloom filter sizes and
sample-to-sub-index assignments that minimise total index size under a
false-positive-rate constraint; `plan` estimates disk and memory requirements
before any build starts; and `registry` federates independently built
sub-indexes into one logical index. No dedicated automation layer of this kind
existed for `kmindex` prior to `kmhelpers`.


# Design and Implementation

## Background: the sub-index structure

A Bloom-filter index stores all BFs of the same size as a single (row-major)
matrix in one file. In such a matrix each column is one BF (one sample) and each
row records the presence (1) or absence (0) of a $k$-mer across all samples
sharing that filter size, assuming a common hash function. A complete index is
therefore a set of such matrices, each one a *sub-index* holding all BFs of a
given size.

The difficulty is that each BF size must match the number of items it stores —
here, the distinct $k$-mers of a sample. If all samples had the same number of
distinct $k$-mers, every BF would share one size and a single matrix would
suffice: the ideal case, where one file is opened per query and one matrix row
gives the answer across all $n$ samples. In practice, sample sizes differ by
several orders of magnitude. Sizing every BF for the largest sample wastes
enormous storage; giving each sample its own single-column matrix minimises
storage but forces a query to open up to $n$ files (potentially millions).
`kmhelpers` takes the middle ground: the user fixes the maximum number of
sub-indexes, and the `profile` step chooses the per-sub-index BF size that
minimises total index size under that constraint.

## The pipeline

`kmhelpers` exposes the index lifecycle as a sequence of commands:

- **`list`** — recursively discovers all samples in a given directory and counts
  each sample's distinct $k$-mers using `ntCard` [@mohamadi2017] (unless the
  counts are provided by the user).
- **`profile`** — determines the best set of sub-index BF sizes given the
  user-defined maximum number of sub-indexes and target false-positive rate.
- **`compose`** — assigns each sample to its sub-index and generates the
  *files-of-files* describing the data origin of each sub-index.
- **`plan`** — validates sample files, available disk space, and memory upfront,
  and emits ready-to-execute pipeline scripts.
- **`apply`** — builds all sub-indexes by invoking `kmindex`, with span-level and
  name-level filtering.
- **`compress`** — optional ZSTD-based block compression of the index
  [@regnier2026].
- **`registry`** — registers several sub-indexes (built locally or anywhere
  accessible) into one logical index, redirecting each query to the relevant
  sub-indexes at query time.

Once an index is built, `kmhelpers` also answers queries (`query`). Multi-step
workflows can be described as declarative YAML pipelines (`pipeline`) and executed
in a single command.

![Overview of the `kmhelpers` workflow. `list` enumerates sample files and
counts $k$-mers; `profile` analyses the count distributions to assign each
sample to a Bloom-filter span and recommend index groupings. Both outputs
feed into `compose`, which generates YAML index definition files. `plan`
then validates sample paths, available disk space and memory, and emits a
ready-to-execute pipeline script; the resulting report is reviewed before
committing to the build. `apply` reads the definition files and invokes
`kmindex` to construct the index. Finally, `query` searches the built index
against user-provided sequences and returns ranked results.](https://notes.inria.fr/uploads/upload_77e9b33f96768c32594e124ee44b0382.png)

<!-- TODO before submission: update the figure to also show index update, and commit the final figure as a local file in the repository (JOSS requires figures committed to the repo, not referenced via an external URL). -->

## Implementation

`kmhelpers` is implemented in Python ($\geq 3.8$) and distributed via Conda, which
installs its bioinformatics dependencies (`kmindex`, `ntCard`) automatically. The
CLI is built with Click [@click], and the package exposes a public Python API
covering all CLI functionality. Together, the declarative YAML index-definition
format and the `plan`/`apply` separation make $k$-mer indexing workflows
accessible to researchers who are not experts in the underlying data structures,
while remaining flexible enough for large-scale production use.


# Tree of Life demonstration

TODO real-world demonstration pending experimental results. Required
content when results arrive: dataset description (number of samples, total
size), hardware used, index parameters chosen automatically by kmhelpers,
output index size, build wall time, and query throughput.


# Availability and documentation

`kmhelpers` is released under the GNU General Public License v3.0 and is available at <https://gitlab.inria.fr/omicfinder/kmhelpers>. The full documentation is available at XXX.


# Acknowledgements

The authors thank Téo Lemane for developing `kmindex` and for his
responsiveness in addressing feature requests and issues raised during the
development of `kmhelpers`. We acknowledge the GenOuest core facility
(<https://www.genouest.org>) for providing the computing infrastructure.
The work was funded by the Inria Challenge "OmicFinder"
(<https://project.inria.fr/omicfinder/>), and by the state funding managed by the French National Research Agency under the France 2030 program [ANR-22-PEAE-0005].


# AI Usage Disclosure

AI assistance was used for constrained tasks (drafting, editing, code suggestions)
under strict human review at every stage. AI provider used: Claude (Anthropic, 2025).


# References
