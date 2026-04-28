# Self-Medicating Social Pain? Bullying and Non-Medical Prescription Opioid Use among Adolescents -- Reproducible Code

This repository contains the analysis code accompanying the manuscript

> **Self-Medicating Social Pain? Bullying and Non-Medical Prescription Opioid Use among Adolescents**
> Mario Morales, MPP; Maeve E. Wallace, PhD; Beth E. Meyerson, PhD; Jennifer S. De La Rosa, PhD
> Health Promotion Sciences, Mel and Enid Zuckerman College of Public Health, University of Arizona, Tucson, AZ

## Submission status

Originally submitted on **March 8, 2026** to *Pediatrics* (American Academy of Pediatrics; manuscript ID **PEDIATRICS/2026/076760**). The manuscript was transferred at the editor's recommendation to *Pediatrics Open Science* (American Academy of Pediatrics; Gold Open Access; Plan S-compliant) under manuscript ID **PEDSOS/2026/001614**, where it is currently under review.

**This is a preprint / submitted version. The manuscript has not yet been accepted, and the code in this repository may be updated in response to peer review.**

## Abstract

The abstract below is reproduced verbatim from the version submitted on 2026-03-08; the wording may change in revision.

**OBJECTIVES.** Many adversities predict adolescent substance use, but it is unknown which create opioid-specific hazard. This study tested whether bullying, suicidality, polysubstance use, risk behavior, sexual victimization, household adversity, and community safety adversity show opioid-specific or general associations with nonmedical prescription opioid use (NMPOU).

**METHODS.** Using 2023 Youth Risk Behavior Survey data (N = 20,103), survey-weighted logistic regressions modeled seven composite predictors. Average marginal effects quantified probability-scale associations. A stacked-outcome test compared associations across five substances (NMPOU, alcohol, marijuana, cigarettes, cocaine). Karlson-Holm-Breen decomposition quantified polysubstance mediation.

**RESULTS.** Predictor-substance associations differed significantly across substance types (p = 0.0075). Bullying was uniquely associated with NMPOU (3.8 pp, 95% CI: 1.5 to 6; AOR = 1.48, 95% CI: 1.18-1.86) and not with any other substance, consistent with an opioid-specific distress pathway. Suicidality was associated with NMPOU (6.2 pp, 95% CI: 4.2 to 8.3; AOR = 1.89, 95% CI: 1.58-2.26) and marijuana but not alcohol, indicating partial specificity. Polysubstance use had the largest overall association (6.3 pp, 95% CI: 4.1 to 8.5; AOR = 2.09, 95% CI: 1.63-2.66) but was not opioid-specific. All seven predictors were independently significant in the fully adjusted model (n = 9,401).

**CONCLUSIONS.** Bullying and suicidality are associated with opioid-specific hazard, indicating why certain adolescents turn to opioids, whereas polysubstance use, sexual victimization, household adversity, and community safety adversity predict substance use broadly. Where bullying or suicidality is identified, targeted opioid prevention is warranted; where NMPOU is present, clinicians should screen for relational victimization and unmet mental health needs.

## Citation

This is a submitted manuscript; cite as a preprint until acceptance.

**APA**

> Morales, M., Wallace, M. E., Meyerson, B. E., & De La Rosa, J. S. (2026). *Self-medicating social pain? Bullying and non-medical prescription opioid use among adolescents* [Manuscript submitted for publication]. Health Promotion Sciences, Mel and Enid Zuckerman College of Public Health, University of Arizona.

**BibTeX**

```bibtex
@unpublished{morales2026nmpou,
  author       = {Morales, Mario and Wallace, Maeve E. and Meyerson, Beth E. and De La Rosa, Jennifer S.},
  title        = {Self-Medicating Social Pain? Bullying and Non-Medical Prescription Opioid Use among Adolescents},
  year         = {2026},
  note         = {Manuscript submitted for publication; under review at Pediatrics Open Science (PEDSOS/2026/001614).},
  institution  = {Health Promotion Sciences, Mel and Enid Zuckerman College of Public Health, University of Arizona},
  url          = {https://github.com/yeridu/nmpou}
}
```

## Data access

This study uses **publicly available, deidentified data** from the 2023 Youth Risk Behavior Survey (YRBS), administered by the United States Centers for Disease Control and Prevention (CDC).

The YRBS file is **not redistributed in this repository.** To reproduce the analysis:

1. Visit <https://www.cdc.gov/yrbs/> and download the **2023 National YRBS Data** in **Microsoft Access (.mdb)** format.
2. The expected filename is `2023_XXH_YRBS_Data.mdb`. Save it to `data/2023_XXH_YRBS_Data.mdb` inside your local clone of this repository.
3. On Windows, install the **Microsoft Access Database Engine 2016 Redistributable** matching the bitness of your R installation (almost always 64-bit). On macOS or Linux, use a Microsoft Access ODBC driver such as the one provided by Actual Technologies, or convert the `.mdb` to a portable format before running.

See `data/README.md` for additional notes.

## Software requirements

- **R >= 4.5.0** (this repository was prepared with R 4.5.2 on Windows 11)
- **RStudio** (recommended) or any R Markdown-compatible IDE
- A working **Microsoft Access ODBC driver** for the `RODBC` package
- A LaTeX or pandoc Word toolchain capable of knitting `.Rmd` to `.docx` (RStudio bundles this by default)

### R packages

The following CRAN package versions were used in the submitted analysis. The same versions are pinned in `renv.lock`.

| Package          | Version  | Purpose                                                              |
|------------------|----------|----------------------------------------------------------------------|
| RODBC            | 1.3.26.1 | Connect to the YRBS Microsoft Access database                        |
| dplyr            | 1.1.4    | Data manipulation                                                    |
| survey           | 4.4.8    | Survey-weighted analyses (svydesign, svyglm, svymean)                |
| psych            | 2.6.1    | Tetrachoric correlations and reliability                             |
| lavaan           | 0.6.20   | Confirmatory factor analysis (WLSMV)                                 |
| knitr            | 1.50     | R Markdown table rendering                                           |
| ggplot2          | 4.0.1    | Forest plots and panel figures                                       |
| flextable        | 0.9.10   | Word-native table rendering                                          |
| officer          | 0.7.2    | Word document construction                                           |
| here             | 1.0.2    | Project-relative file paths                                          |
| patchwork        | 1.3.2    | Composite figure layouts                                             |
| eulerr           | 7.0.4    | Proportional Euler diagrams (Figure 3, Figure S1)                    |
| marginaleffects  | 0.32.0   | Average marginal effects on the probability scale (Allison delta fix)|
| mitools          | 2.4      | Multiple-imputation pooling helpers                                  |
| rmarkdown        | 2.30     | Knit `.Rmd` to Word                                                  |

A plain-text `sessionInfo()` snapshot is provided in `sessionInfo.txt` for reviewers who prefer to install packages manually.

## Reproduction steps

```bash
# 1. Clone the repository
git clone https://github.com/yeridu/nmpou.git
cd nmpou

# 2. Download the 2023 YRBS .mdb from the CDC and save it as
#    data/2023_XXH_YRBS_Data.mdb (see data/README.md)

# 3. Open nmpou.Rproj in RStudio (or set the working directory to the repo root)

# 4. Restore the pinned package environment
#    install.packages("renv")
#    renv::restore()

# 5. Knit the analysis script
#    rmarkdown::render("nonmedicaluse_painmed_script_v4.Rmd")
```

The knit produces a Word document containing all R code, intermediate output, and tables. Generated CSV outputs land in `output/` and `output/supplement_tables/`; generated TIFFs and Word table files land in `figures/` and `tables/`.

## Repository structure

```
nmpou/
|-- nonmedicaluse_painmed_script_v4.Rmd    # Analysis script (4,049 lines, 583 expressions)
|-- README.md                              # This file
|-- LICENSE                                # MIT license for the code
|-- nmpou.Rproj                            # RStudio project file
|-- renv.lock                              # Pinned R package versions
|-- sessionInfo.txt                        # Plain-text sessionInfo() snapshot (fallback)
|-- .here                                  # Sentinel for the here package
|-- .gitignore
|-- .gitattributes
|-- data/
|   |-- README.md                          # CDC data download instructions
|   `-- .gitkeep                           # The .mdb is NOT in this repo
|-- output/.gitkeep                        # CSV outputs land here on reproduction
|-- tables/.gitkeep                        # CSV intermediates land here on reproduction
`-- figures/.gitkeep                       # TIFFs and Word tables land here on reproduction
```

## Outputs produced

When the analysis script is rendered, it generates the following files (relative to the repository root):

| Output file                                                  | Manuscript element                |
|--------------------------------------------------------------|-----------------------------------|
| `figures/Table1_Prevalence.docx`                             | Table 1 (main text)               |
| `output/table1_data.csv`                                     | Underlying data for Table 1       |
| `figures/Figure1_ForestPlot_M5.tiff`                         | Figure 1                          |
| `figures/Figure2_SubstanceSpecificity.tiff`                  | Figure 2                          |
| `figures/Figure3_FrameworkEndorsement.tiff`                  | Figure 3                          |
| `figures/FigureS1_FrameworkEndorsement_BySex.tiff`           | Figure S1                         |
| `output/supplement_tables/TableS1.csv` ... `TableS15.csv`    | Tables S1--S15 (supplement)       |
| `figures/TableS4_CompositeSummary.docx` ... `TableS15_*.docx`| Word-formatted supplement tables  |
| `tables/Table_S5_AME_by_substance.csv`                       | AME intermediate (Table S5)       |
| `tables/AME_heterogeneity_tests.csv`                         | Heterogeneity intermediates       |
| `tables/AME_stacked_by_substance.csv`                        | Stacked-model intermediates       |
| `tables/LPM_stacked_sensitivity.csv`                         | LPM sensitivity intermediates     |
| `tables/stacked_relaxed_constraint.csv`                      | Stacked relaxed-constraint check  |
| `tables/KHB_mediation_CIs.csv`                               | KHB mediation intermediate        |

## Generative AI disclosure

The following disclosure is reproduced verbatim from the Acknowledgments section of the submitted manuscript:

> "MM used generative artificial intelligence (Claude Sonnet, Anthropic) to assist with manuscript editing, code documentation, and formatting for journal guidelines. MM takes full responsibility for the integrity and accuracy of these elements."

## License

The code in this repository is released under the **MIT License** (see `LICENSE`). This license applies to the code only. The article itself, once accepted and published, will be licensed separately by the journal under its open-access policy.

## Contact

Corresponding author: **Mario Morales, MPP** -- <mariomorales@arizona.edu>
1295 N. Martin Ave., Tucson, AZ 85724
