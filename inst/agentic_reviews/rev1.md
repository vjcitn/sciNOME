# Review of `sciNOME` with respect to Bioconductor standards, scientific clarity, S4 classes, and testing

Based on a static review of the repository, the package has a good skeleton for Bioconductor, but it is not yet strong on Bioconductor conventions, scientific clarity, or formal class design.

## Overall assessment

- **Strengths:** roxygen-based man pages exist, `biocViews` is set, vignettes use `BiocStyle` and end with `sessionInfo()`, and `testthat` is present (`DESCRIPTION:21-64`, `vignettes/*.Rmd`, `tests/testthat.R:1-12`).
- **Main concerns:** custom S3 object instead of Bioconductor-style S4/container classes, documentation inconsistencies, limited biological interpretation in vignettes, and tests focused mostly on happy paths.

## 1. Bioconductor standards

### Positive points

- `DESCRIPTION` includes `biocViews`, `URL`, `BugReports`, and `Suggests: BiocStyle, testthat, knitr, rmarkdown` (`DESCRIPTION:19-21,54-59`).
- Vignettes are lightweight and likely buildable because they use simulated data (`vignettes/RNA_Analysis.Rmd:35-55`, `vignettes/Epi_Analysis.Rmd:36-75`).

### Problems

- **License inconsistency:** `DESCRIPTION` says `GPL-3`, but `README.md` says `MIT License` and points to a `LICENSE` file that does not exist (`DESCRIPTION:18`, `README.md:221-223`).
- **Installation instructions are misleading:** `BiocManager::install("Medinfo-Lab/sciNOME")` is presented as GitHub installation, which is not standard (`README.md:7-15`).
- **Repository hygiene risk:** the repository root contains `.Rhistory` and `.Renviron`, while `.Rbuildignore` only excludes `.Rproj` artifacts (`.Rbuildignore:1-2`).
- `NEWS.md` still says “Initial CRAN submission,” which is out of sync with a Bioconductor-facing package (`NEWS.md:1-3`).
- No package-level help page or `CITATION` file was found.

## 2. Scientific clarity of documentation and vignettes

### Positive points

- The vignettes are structured as workflows and are approachable for new users.
- They explain required inputs at a practical level.

### Problems

- The documentation is workflow-oriented but not sufficiently interpretive from a scientific perspective. It shows what to run, but not enough of:
  - why CpG and GpC are biologically distinct,
  - why promoter/region aggregation is the right abstraction,
  - what assumptions are made when linking regions to genes,
  - when each integration mode should be preferred.
- There are clear errors in the README that weaken confidence:
  - CpG aggregation uses `gpc_dir` instead of `cpg_dir` (`README.md:129-137`).
  - GpC differential analysis still uses `CpG_level`, `CpG_meth`, and `CpG_nonmeth` names (`README.md:180-188`).
  - Some inline comments also confuse CpG and GpC semantics (`README.md:156-160`).
- The plotting vignette functions more as a gallery than as a guide to scientific interpretation (`vignettes/Plot.Rmd:26-194`).

## 3. Use of S4 classes

This is the weakest area relative to Bioconductor expectations.

- No `setClass`, `setMethod`, or validity methods were found under `R/`.
- The core object is an S3 list-based object with `class(object) <- "RNA"` (`R/rna.R:64-85`).
- Tests also treat it as an S3 class (`tests/testthat/test-rna.R:14-16`).

### Why this matters

Bioconductor strongly favors interoperable S4 containers such as `SummarizedExperiment`, `SingleCellExperiment`, `GRanges`, or `MultiAssayExperiment`. The current reliance on base `list` and `data.frame` structures weakens interoperability and omits formal validity checks.

### Recommendation

- At minimum, move the RNA object to a formal S4 class with validity checks.
- Preferably, build on existing Bioconductor containers instead of maintaining a package-specific object system.

## 4. Testing

### Positive points

- Tests exist for RNA, epigenetics, integration, and plotting (`tests/testthat/`).
- Tests use small synthetic data, which is appropriate for package checks.

### Limitations

- Coverage is mostly happy-path only:
  - RNA tests cover one end-to-end path (`tests/testthat/test-rna.R:3-39`).
  - Multi-omics tests cover only one tri-omics scenario (`tests/testthat/test-multi_omics.R:3-40`).
  - Plot tests mostly assert returned class, not content or failure handling (`tests/testthat/test-plot.R:20-47`).
- Important missing tests include:
  - invalid metadata or sample matching,
  - absent mitochondrial genes,
  - empty or overfiltered matrices,
  - unsupported parameter values,
  - all integration modes,
  - optional differential-result branches,
  - error and warning behavior.

## Priority recommendations

1. Fix documentation correctness first: license, installation instructions, CpG/GpC mistakes, and outdated `NEWS.md`.
2. Adopt formal S4 or standard Bioconductor container classes for core data structures.
3. Strengthen the scientific narrative in the vignettes, especially assumptions, interpretation, and intended use cases.
4. Expand tests to cover failure modes and branch-specific behavior.
