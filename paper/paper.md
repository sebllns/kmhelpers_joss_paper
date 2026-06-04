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

Large-scale genomic sequence search has become a central challenge in modern
bioinformatics [REFSTODO]. Most scalable methods use Bloom filters (BF) [REFTODO]. At query time, this enables assigning the queried $k$-mers to the set of datasets they belong to. Building such indexes on a vast set of genomic samples requires several steps. First, determine the length of each sample (in terms of its number of distinct $k$-mers). Second, given a user-defined false positive rate, distribute input samples into distinct groups, each BF of a group having the same fixed size. The sizes of BF of groups have to be optimized to reduce the final whole index size. Third, given computational constraints, the whole construction is split into jobs, each generating sub-parts of the index, and a final step merges these sub-parts. 

Those steps require expertise, and any error at any stage can invalidate downstream results and waste significant
computation time. 

[//]: # $k$-mer indexes built on Bloom filters allow efficient querying of large sequence collections. However, building and managing such indexes in practice involve a complex chain of steps. 

[//]: # such as those produced by `kmindex` [@lemane2024], : enumerating sample files, counting $k$-mers, selecting appropriate Bloom-filter parameters,
generating index definitions, orchestrating build jobs, and compressing outputs for long-term storage. Each step requires expertise, and errors at any stage can invalidate downstream results and waste significant computation time.

We propose `kmhelpers`, an open-source Python toolkit that automates this entire workflow for building (and querying) indexes created using `kmindex` [@lemane2024]. `kmhelpers` provides both a command-line interface (CLI) and a Python API, covering every stage of the $k$-mer index lifecycle.

- sample discovery and $k$-mer counting (`list`),
- Bloom-filter span profiling and grouping (`profile`),
- index definition generation (`compose`),
- upfront validation of sample files, available disk space and memory with
  generation of ready-to-execute pipeline scripts (`plan`),
- index building from definition files (`apply`),
- sequence querying (`query`),
- ZSTD-based index (block) compression (`compress`) [@regnier2026],
- registry management (`registry`).

Multi-step workflows can be described as declarative YAML pipelines
(`pipeline`) and executed in a single command.


# Statement of Need

$k$-mer based sequence databases built with tools like `kmindex` and `kmtricks`
[@lemane2022] are used to address large-scale questions in
genomics, metagenomics, and population studies. However, the gap between having
the raw indexing software and running a reproducible, end-to-end indexing
workflow is significant. Researchers face recurring pain points:

**Parameter selection.** Bloom filter-based indexes require choosing a filter
size (or *span*) per sample. Choosing this parameter incorrectly leads to
either storage consumption or unacceptably high false-positive rates.
`kmhelpers` automates this with its `profile` command, which reads $k$-mer count
distributions produced by `ntCard` [@mohamadi2017] and assigns each sample to
the smallest span that keeps the false-positive rate below a user-defined
threshold.

**Workflow orchestration.** Building a large index may involve hundreds to millions of samples spread across multiple sub-indexes. Managing
these build jobs manually is error-prone. `kmhelpers` introduces a declarative index definition format (YAML) and two complementary commands: `plan`, which validates sample files, available disk
space and memory upfront and emits ready-to-execute pipeline scripts; and
`apply`, which reads the definition files and runs the build, with support
for span-level and name-level filtering.

Together, these features make $k$-mer indexing workflows accessible to
researchers who are not experts in the underlying data structures, while
remaining flexible enough for large-scale production use.

# State of the Field

Several tools address large-scale sequence search using $k$-mer based data
structures. BIGSI [@bradley2019] and COBS [@bingmann2019] use compressed Bloom
filter matrices to answer presence/absence queries across collections of
sequencing experiments. HowDeSBT [@harris2019] and Mantis [@pandey2018] use
sequence Bloom trees and counting quotient filters, respectively, to support
similar queries. `kmindex`, which `kmhelpers` wraps,
builds on the `kmtricks` counting pipeline and is designed for efficient querying of large sequence collections.

`kmhelpers` does not compete with any of these approaches at the algorithmic
level. Its contribution is orthogonal: it provides the workflow automation,
parameter optimisation, and index lifecycle management that are necessary to
deploy `kmindex`-based databases in practice, but are absent from the core
tool. Similar automation layers exist for other complex bioinformatics
pipelines (e.g., Snakemake [@koster2012] workflows wrapping alignment or
assembly tools), but no dedicated automation layer existed for `kmindex`
prior to `kmhelpers`.

# Design and Implementation

![Overview of the `kmhelpers` workflow. `list` enumerates sample files and
counts $k$-mers; `profile` analyses the count distributions to assign each
sample to a Bloom-filter span and recommend index groupings. Both outputs
feed into `compose`, which generates YAML index definition files. `plan`
then validates sample paths, available disk space and memory, and emits a
ready-to-execute pipeline script; the resulting report is reviewed before
committing to the build. `apply` reads the definition files and invokes
`kmindex` to construct the index. Finally, `query` searches the built index
against user-provided sequences and returns ranked results.](https://notes.inria.fr/uploads/upload_77e9b33f96768c32594e124ee44b0382.png)

[[TODO]]

`kmhelpers` is implemented in Python (≥3.8) and distributed via Conda with
automatic installation of its bioinformatics dependencies (`kmindex`,
`ntCard`). The CLI is built with Click [@click] and the package exposes a
public Python API covering all CLI functionality. 





# Acknowledgements

The authors thank Téo Lemane for developing `kmindex` and for his
responsiveness in addressing feature requests and issues raised during the
development of `kmhelpers`. We acknowledge the GenOuest core facility
(<https://www.genouest.org>) for providing the computing infrastructure.
The work was funded by the Inria Challenge "OmicFinder"
(<https://project.inria.fr/omicfinder/>).

# AI Usage Disclosure

AI assistance was used for constrained tasks (drafting, editing, code suggestions)
under strict human review at every stage. AI provider used: Claude (Anthropic, 2025).

# References
