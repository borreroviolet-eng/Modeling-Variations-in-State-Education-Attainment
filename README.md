# Modeling Variations in State Education Attainment

A multiple regression analysis of state-level factors associated with bachelor's degree attainment across the 50 U.S. states.

**Violet Borrero** · Seminar in Applied Mathematics (MAT 430), Southern New Hampshire University · December 2023

## Research question

What state-level characteristics are associated with differences in educational attainment across the United States?

While exploring the 2022 NAEP results, I noticed that student outcomes varied sharply by where students lived. The gap was also visible in attainment rates, especially at the college/no-college threshold. This project models that variation using publicly available state data.

## Data

I compiled a dataset of **50 states × 28 variables**, guided by Boissiere's (2004) framework of education inputs: resources, teachers, institutional management, and social context. Washington, D.C. was excluded as an outlier in nearly every category.

| Category | Example variables | Source |
| --- | --- | --- |
| Outcomes | Bachelor's and high school attainment, 2022 NAEP Grade 8 reading and math | U.S. Census, NAEP (Nation's Report Card) |
| Spending | Per capita total, K–12, and higher-ed spending; Title I funds | Tax Policy Center, U.S. Census |
| Teachers | Normalized teacher pay index, pupil–teacher ratio, certification exam requirements | NCES (EDGE, ELSI, State Education Practices) |
| Early education | Kindergarten offering and attendance requirements, minimum instructional hours | NCES State Education Practices |
| Context | Poverty rate, Democratic-to-Republican vote ratio (2016, 2020) | U.S. Census / CRS, FEC |

Descriptions and sources for every variable are in `data/MAT430_Education_variable_dictionary.csv`. In the data, `-1` marks a value the source didn't report.

## Methods

All analysis was done in R (RStudio). The original work ran in stages across several notebooks. In 2026, I consolidated it with help from Claude (Anthropic's AI assistant) into a single notebook, `state_education_attainment_analysis.Rmd`, which reproduces every figure in the paper. The originals are kept in `archive/`.

1. **Correlation screening.** Grouped related variables and used `pairs()` plots to spot strong relationships, collinearity, and outliers.
2. **Model comparison.** Tested logistic regressions that classified whether states met the national NAEP average, and linear regressions on attainment. The linear models performed far better, and bachelor's attainment was the most responsive outcome.
3. **Model building.** Added predictors one at a time and compared each step with nested F-tests (`anova()`) and three R² measures (multiple, adjusted, predictive) to guard against overfitting with a small sample.
4. **Diagnostics.** Checked residual histograms, residuals vs. fitted values, a Normal Q-Q plot, and variance inflation factors (`vif()`).

| Model | Predictors | Adjusted R² | Predictive R² | RSE |
| --- | --- | --- | --- | --- |
| lm1 | poverty rate | 0.562 | 0.534 | 3.48 |
| lm12 | + teacher pay | 0.713 | 0.688 | 2.81 |
| lm123 | + pupil–teacher ratio | 0.731 | 0.694 | 2.72 |
| lm1234 | + political lean | **0.799** | **0.769** | **2.36** |

## Results

The final model explains about **80% of the variation** in states' bachelor's attainment rates (adjusted R² = 0.80, F = 49.6, p < 0.001):

```
predicted % with bachelor's = 30.32 − 0.86(poverty rate) + 11.68(teacher pay index)
                              − 0.18(pupil–teacher ratio) + 3.99(D/R vote ratio)
```

- **Poverty rate** had the strongest and most consistent association (p < 0.001).
- **Pupil–teacher ratio** was negatively associated with attainment (p = 0.05).
- **Teacher pay** is a normalized labor-cost index, not raw salary, and fell just short of significance (p = 0.053). It should not be read as "raise salaries and attainment rises." Research by Hanushek and others finds weak returns to spending alone.
- **Political lean** stands in for policy priorities and social context. It is not a lever a state can pull.
- All VIFs were below 2.5, so collinearity was within a conservative threshold.

## Limitations

- **Associations, not causes.** A cross-sectional model of 50 states cannot show that any factor *drives* attainment, and unobserved confounders are likely.
- **Small sample, single snapshot.** Results reflect one reporting year and were not tested on other time periods.
- **Data timing mismatches.** Some policy variables, such as 2017–18 kindergarten requirements, don't line up exactly with the student cohorts measured.
- **Missing "software" inputs.** Curriculum and pedagogy are decided locally and aren't tracked in state-level databases.

### Looking back

Rereading this a few years later, I'd tighten the causal language. Phrases like "drivers" should read "factors associated with." I'd also revisit the logistic models: an AUC below 0.5 is worse than chance, which usually signals a coding or label issue rather than a weak model. Natural next steps are a sensitivity analysis (e.g., R's `sensemakr`), principal component analysis of the teacher-quality and early-education factors, and testing the model on other years.

## Repository contents

```
├── README.md
├── state_education_attainment_analysis.Rmd              # consolidated analysis (start here)
├── state_education_attainment_analysis.html             # rendered results
├── Modeling_Variations_in_State_Education_Attainment.pdf  # final written report
├── archive/                                             # original 2023 working notebooks
│   ├── education_notebook_v3.Rmd
│   ├── education_notebook_v3.nb.html
│   ├── disaster101.Rmd
│   └── disaster101.nb.html
└── data/
    ├── MAT430_Education.xlsx                        # original workbook
    ├── MAT430_Education_all_data_sans_dc.csv        # modeling dataset (50 states)
    └── MAT430_Education_variable_dictionary.csv     # variable descriptions and sources
```

## Key references

- Boissiere, M. (2004). *Determinants of Primary Education Outcomes in Developing Countries.* World Bank.
- Bradley, S., Green, C., & Hanushek, E. (2020). Education production functions. In *The Economics of Education* (pp. 161–170). Academic Press.
- Dickinson, E. E. (2016). Coleman report set the standard for the study of public education. *Johns Hopkins Magazine.*

## Licenses
Code is released under the [MIT License](LICENSE). The paper and
documentation are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Data in `data/` was compiled from public sources (NCES, NAEP, U.S. Census,
FEC, Tax Policy Center) and remains subject to their original terms.
