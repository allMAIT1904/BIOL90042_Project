# Parkinson's Disease Research Program
## BIOL90042 Final Project — Group 7

## Overview

This project analyses four connected arms of a simulated Parkinson's disease
research program:

1. Population genetics.
2. RNA sequencing of patient-derived neurons.
3. High-throughput drug screening.
4. A clinical trial comparing a new compound with L-DOPA.

The analyses are compiled in `compiled.qmd`, which combines R code,
visualisations, statistical testing, interpretation, and responses to the
project questions.

An integration section links genetic risk, gene expression, compound targets,
and clinical outcomes using the identifiers available in the supplied datasets.

## Project files and required data

The compiled report expects the following files:

```text
BIOL90042_Project/
├── README.md
├── compiled.qmd
└── data/
    ├── prs.Rda
    ├── rnaseq.Rda
    ├── rnaseq_gene_info.Rda
    ├── drugscreen.Rda
    └── gait.Rda
```

This layout describes the paths required by `compiled.qmd`, rather than a
complete inventory of the repository.

| File | Purpose |
|------|---------|
| `compiled.qmd` | Combined analysis, interpretation, and project questions |
| `data/prs.Rda` | Participant genotypes and GWAS effect estimates for polygenic risk scoring |
| `data/rnaseq.Rda` | RNA-seq counts and sample metadata |
| `data/rnaseq_gene_info.Rda` | Gene identifiers, gene names, and supplied Gene Ontology annotations |
| `data/drugscreen.Rda` | Plate luminescence, well-level compound identities, and compound target annotations |
| `data/gait.Rda` | Participant orientation measurements and clinical-trial metadata |

## Analyses

### 1. Population genetics

This section calculates polygenic risk scores (PRS) for 300 participants:
150 Parkinson's disease cases and 150 controls.

The workflow includes:

- Reshaping genotype data across 15 loci.
- Joining allele dosages to GWAS beta coefficients.
- Calculating each participant's PRS as the sum of dosage × beta.
- Identifying participants with the highest and lowest scores.
- Comparing PRS distributions using descriptive statistics and plots.
- Assessing normality and differences in group variability.
- Comparing groups using Welch's t-test, with a Wilcoxon rank-sum
  sensitivity analysis.

### 2. RNA-seq of patient-derived neurons

This section compares gene expression between Parkinson's disease cases
and healthy controls across 16 samples.

The workflow includes:

- Inspecting counts and sample metadata.
- Filtering lowly expressed genes.
- Applying TMM normalisation.
- Constructing the design matrix and case-versus-control contrast.
- Fitting a limma-voom model with empirical Bayes moderation.
- Identifying differentially expressed genes.
- Visualising results using MA and volcano plots.
- Joining differentially expressed genes to supplied GO annotations.
- Examining sample relationships using MDS and a top-gene heatmap.

The GO analysis summarises supplied annotations for differentially expressed
genes; it is not presented as a formal GO enrichment analysis.

### 3. Drug screening

This section analyses a 1,536-well screen of 192 compounds in healthy
donor-derived dopaminergic neurons exposed to MPP.

Each active compound is compared with a chemically matched inactive control,
with four technical replicate wells per condition.

The workflow includes:

- Converting plate measurements and metadata into tidy format.
- Checking plate dimensions, replicate counts, and compound annotations.
- Visualising luminescence distributions and spatial patterns.
- Estimating mean test-minus-control differences and fold changes.
- Applying one-sided Welch's t-tests.
- Controlling the false discovery rate using the Benjamini–Hochberg procedure.
- Running Wilcoxon rank-sum tests as a sensitivity analysis.
- Annotating screening hits with their supplied targets and mechanisms.

A screening hit is defined as a positive mean RLU difference with
BH-adjusted p < 0.05.

Luminescence is treated as a proxy for neuronal viability. Hits are
plate-level candidates, not confirmed neuroprotective compounds.

### 4. Clinical trial

This section compares gait measurements from 16 participants receiving
either L-DOPA or a new compound.

The workflow includes:

- Reshaping orientation measurements into long format.
- Plotting participant orientation over time.
- Calculating descriptive statistics for each participant.
- Selecting orientation standard deviation as the gait-variability outcome.
- Comparing treatment groups using a Wilcoxon rank-sum test.
- Investigating age and exercise using linear models.
- Evaluating treatment associations after adjustment for both covariates.

Lower orientation standard deviation is interpreted as less variable,
more stable gait within this analysis.

### 5. Integration across study arms

The integration section addresses:

1. Participant overlap across the study arms.
2. Whether PRS, age, treatment, or exercise align with RNA-seq sample structure.
3. Overlap between drug-hit targets and differentially expressed genes.
4. Associations between PRS and gait variability.
5. Overall findings, limitations, and future research priorities.

Participant-level joins use `donorid`. Gene-level joins use Entrez IDs,
with HGNC symbols providing readable labels.

Exact four-arm donor overlap cannot be established because the drug-screen
dataset does not supply a donor identifier.

## Software and packages

The document uses R and Quarto.

Packages referenced in the current document include:

- CRAN: `tidyverse`, `pheatmap`, `ggrepel`, `broom`, `patchwork`,
  `ggbeeswarm`, `knitr`, `here`, and `rstatix`.
- Bioconductor: `edgeR`, `limma`, and `RNAseq123`.

### Install dependencies

Run this setup code once before rendering. It installs only missing packages.

```r
cran_packages <- c(
  "tidyverse", "pheatmap", "ggrepel", "broom", "patchwork",
  "ggbeeswarm", "knitr", "here", "rstatix"
)

missing_cran <- cran_packages[
  !vapply(cran_packages, requireNamespace, logical(1), quietly = TRUE)
]

if (length(missing_cran) > 0) {
  install.packages(
    missing_cran,
    repos = "[https://cloud.r-project.org](https://cloud.r-project.org)"
  )
}

bioc_packages <- c("edgeR", "limma", "RNAseq123")

missing_bioc <- bioc_packages[
  !vapply(bioc_packages, requireNamespace, logical(1), quietly = TRUE)
]

if (length(missing_bioc) > 0) {
  if (!requireNamespace("BiocManager", quietly = TRUE)) {
    install.packages(
      "BiocManager",
      repos = "[https://cloud.r-project.org](https://cloud.r-project.org)"
    )
  }

  BiocManager::install(
    missing_bioc,
    ask = FALSE,
    update = FALSE
  )
}
```

Load `edgeR` and `limma` explicitly in the shared setup because the RNA-seq
analysis calls functions from both packages:

```r
library(edgeR)
library(limma)
```

## Running the report

1. Place all five `.Rda` files in the `data/` directory.
2. Install the required packages.
3. Review the reproducibility notes below.
4. Open `compiled.qmd` and render it with Quarto.

From the project root:

```bash
quarto render compiled.qmd
```

The document is configured to produce an HTML report with:

- A table of contents extending to level-three headings.
- Numbered sections.
- Visible code and code tools.
- Embedded resources.
- Paged data-frame output.

The final section calls `sessionInfo()` to record the R environment used
for the analysis.

## Document formatting conventions

Use the following structure consistently for each main analysis topic:

- `# Topic heading`: for example, `# Clinical trial`.
- `## Introduction`.
  - `### Background`.
  - `### Aims`.
- `## Code and interpretation`.
  - Use `###` headings for individual analysis components.
  - Use bold text for further sub-subheadings.
- `## Questions`.
  - Use `### Question 1.`, `### Question 2.`, and so on.
  - State each question in italics immediately below its heading.
  - Follow with the answer, supporting code, and interpretation.
- End each topic with a horizontal rule (`---`).

### Topic template

# Topic heading

## Introduction

### Background

Introduce the scientific context and rationale.

### Aims

State the objectives of the analysis.

## Code and interpretation

### Data preparation

Describe the input data and preprocessing.

```{r}
# Data preparation code
```

**Interpretation**

Explain the relevant outputs.

### Statistical analysis

Describe the analysis, assumptions, and results.

```{r}
# Statistical analysis code
```

## Questions

### Question 1.

*State the first question here.*

Provide the answer and supporting interpretation.

### Question 2.

*State the second question here.*

Provide the answer and supporting interpretation.

---

## Visual conventions

The shared setup defines the following colour scheme:

| Colour | Intended meaning |
|--------|------------------|
| Blue | Healthy controls, matched controls, or L-DOPA control |
| Orange | Parkinson's disease cases, test compounds, or the new compound |
| Green | Statistically supported results or exercise |
| Grey | Background or non-significant observations |

Some plots apply their own scales, including continuous gradients for
luminescence, PRS, and age.

## Reproducibility notes for the current draft

The current `compiled.qmd` requires review before a clean-session render.

1. Package installation occurs inside the shared setup.
   Consider moving installation to a separate setup step to avoid reinstalling
   packages during rendering.

2. The shared setup installs `edgeR` and `limma` but does not explicitly
   attach them. Add `library(edgeR)` and `library(limma)`.

3. `RNAseq123`, `here`, and `rstatix` are loaded later in the document but
   are not included in its initial installation block.

5. The drug-screen answer references `row_model` and `col_model`, but their
   definitions are not present in the uploaded document. Define these models
   before that answer chunk.

6. Some headings lack a space after the Markdown heading marker, including
   parts of the integration section. Standardise these headings and add an
   explicit level-one clinical-trial heading.

Render from a fresh R session to check that all objects are created within
the document rather than inherited from an existing workspace.

## Limitations

- The RNA-seq and clinical-trial cohorts are small.
- Drug-screen replicates are technical wells from one healthy donor.
- Luminescence may reflect metabolic activity as well as viable cell number.
- Transcript abundance does not establish protein activity or drug-target engagement.
- Orientation variability captures only one aspect of Parkinson's disease-related impairment.
- Cross-dataset associations and target overlaps are exploratory.
- Compound mechanisms and GO categories are taken from supplied annotations.

Future work should prioritise independent donor validation, dose-response
experiments, orthogonal viability assays, target-engagement measurements,
and larger cohorts with pre-specified statistical models.

## Authors

Jesamine Newman, Nikki Leung, Tushar Raja, Yihan Yan
