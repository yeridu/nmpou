---
output:
  word_document:
    reference_docx: "C:/Users/yerid/Documents/reference_doc.docx"
---

**Understanding the KHB Decomposition**

**1. The question this test answers**

The stacked-outcome test showed that bullying is linked specifically to opioids. KHB answers a natural follow-up question:

**Of a risk factor's link to opioids, how much is direct, and how much is carried by general substance use?**

The idea is simple. A bullied adolescent may be more likely to use opioids for two different reasons: (a) a direct path from bullying to opioids, and (b) an indirect path, because bullied adolescents tend to use more of every substance, and opioids are part of "every substance." A risk factor that is truly opioid-specific should be mostly direct -- only a small share should run through general substance use.

**2. The intuition: shared with general use, or its own**

When bullying is linked to opioid use, that link can come from two sources. Part of it may be shared with general substance use: bullied adolescents tend to use more substances of all kinds, and opioid use usually appears together with that general use, so part of the bullying-opioid link is really just this overlap. The rest is bullying's own link to opioids, separate from general use.

KHB splits the link between these two sources and reports how large the shared part is. For bullying, only about a tenth of the link is the shared part, so almost all of it is bullying's own -- consistent with opioid specificity. For sexual victimization, about a quarter is the shared part, so more of its link travels with general substance use -- consistent with broad rather than opioid-specific risk.

**3. Why you cannot just compare two simple models (the trap)**

The obvious approach would be to fit two logistic models -- one without polysubstance use (the "total" effect) and one with it (the "direct" effect) -- and read the shrinkage of the risk factor's coefficient as the indirect, mediated part. This works in ordinary linear regression. In logistic regression it does not, because of a property called noncollapsibility.

In plain words: in logistic regression, simply adding any variable changes the scale of all the coefficients (the model's built-in amount of unexplained variation is fixed, so adding a variable rescales everything). As a result, a coefficient can shrink when you add polysubstance use even if there is no real mediation at all -- a false impression of mediation. So the naive two-model comparison can mislead.

**4. The KHB fix (residualizing the mediator)**

KHB removes the trap by making the two models contain the **same number of variables**, so they are on the **same scale** and can be compared cleanly. It does this with a trick: instead of dropping polysubstance use, it replaces it with the **part of polysubstance use that is not explained by the risk factor** (its "residual").

Because the residual carries no information about the risk factor, putting it in the model lets the risk factor's coefficient absorb its full (total) effect; putting the real polysubstance measure in instead gives the direct effect. Both models have the same number of variables, so the difference between the two coefficients is a clean mediation estimate, free of the rescaling artifact.

**5. The three steps (with the code and variable names)**

Using bullying (bu_1) as the example, the outcome is NMPOU (q49_1) and the mediator is lifetime polysubstance use (su_life, with a 0/1 numeric version su_life_num for the linear step).

Step 1 -- explain the mediator. Regress the mediator on the risk factor plus all the other risk factors and demographics, using survey-weighted **linear** regression, and save the residuals. In the script:

```r
stage1_bu <- svyglm(su_life_num ~ bu_1 + rb_1 + sc_1 + sv_1 + q2_1 + q3_1 + q4_1,
                    design = des_khb, family = gaussian())
resid_su_bu <- residuals(stage1_bu)
```

(The linear, gaussian, step is deliberate: it guarantees the residuals are uncorrelated with the risk factor, which is what the method needs; this holds even though the mediator is yes/no.)

Step 2a -- the direct effect. Fit the full logistic model that includes the real polysubstance measure; the coefficient on the risk factor is the direct effect:

```r
fit_direct_bu <- svyglm(q49_1 ~ bu_1 + su_life + rb_1 + sc_1 + sv_1 + q2_1 + q3_1 + q4_1,
                        design = des_khb, family = quasibinomial())
```

Step 2b -- the total effect. Fit the same logistic model but with the residuals in place of the real polysubstance measure; the coefficient on the risk factor is the total effect:

```r
fit_total_bu <- svyglm(q49_1 ~ bu_1 + resid_su_bu + rb_1 + sc_1 + sv_1 + q2_1 + q3_1 + q4_1,
                       design = des_khb, family = quasibinomial())
```

Then the decomposition (on the log-odds scale):

```r
indirect_bu <- beta_total_bu - beta_direct_bu
prop_med_bu <- (indirect_bu / beta_total_bu) * 100   # percent mediated
```

**6. A worked example**

Suppose bullying's total log-odds coefficient is 0.50 and its direct coefficient is 0.45. Then the indirect (mediated) part is 0.50 - 0.45 = 0.05, and the percent mediated is 0.05 / 0.50 times 100 = 10 percent. In this study bullying's mediated share was about 10.3 percent -- the rest of its association is direct.

**7. What the decomposition found**

Lower percent mediated means more of the association is direct, which is the signature of a more opioid-specific risk factor.

| Risk factor | Share carried by general polysubstance use | Reading |
|-------------|---------------------------------------------|---------|
| Bullying | 10.3% | mostly direct -- opioid-specific |
| Community safety adversity | 14.2% | mixed |
| Suicidality | 15.3% | largely direct |
| Risk behavior | 18.4% | mixed |
| Household adversity | 21.8% | more general |
| Sexual victimization | 25.2% | most carried by general use -- broad |

Confidence intervals for these shares come from a design-based bootstrap that resamples primary sampling units within strata, which respects the survey design.

**8. How to read the numbers honestly**

- This is an accounting decomposition, not causal mediation. With cross-sectional data we cannot claim that polysubstance use causally transmits the effect of bullying on opioids. KHB tells us what fraction of the risk factor's log-odds is statistically accounted for by its shared association with polysubstance use. A causal mediation claim would require temporal ordering (the mediator before the outcome) and no unmeasured confounders of the mediator-outcome relationship.
- The percent mediated is scale-free. Because both models share the same scale, the unexplained-variation term cancels out of the ratio, so the percent mediated is not distorted by the logistic-scale problem described in Section 3.

**9. How this relates to the source papers**

The method comes from Karlson, Holm, and Breen (2012), with the total-direct-indirect interpretation formalized by Breen, Karlson, and Holm (2013); both are cited below. The reason the naive two-model comparison fails (Section 3) is the same logistic scale problem that Allison (1999) raised for comparing coefficients across groups in the companion stacked-outcome method. KHB's residualization puts the two models on a common scale, which is why its percent mediated is scale-free and does not require estimating any noise-ratio correction.

**10. Where this lives in the code**

All steps above are in the analysis script (nonmedicaluse_painmed_script_v4.Rmd), in the KHB chunk:

- the mediator is su_life (numeric su_life_num); the outcome is q49_1 (NMPOU);
- Step 1 fits stage1_bu, stage1_sc, and so on, and saves residuals (resid_su_bu, resid_su_sc, and so on);
- the direct effects are read from the full models (fit_direct_bu for the full-sample predictors, fit_direct_hh for the national-module predictors household and community safety);
- the total effects are fit_total_bu, fit_total_sc, and so on;
- the mediated shares are prop_med_bu, prop_med_sc, prop_med_sv, prop_med_rb, prop_med_hh, prop_med_ms.

Data: 2023 Youth Risk Behavior Survey (CDC), publicly available at https://www.cdc.gov/yrbs/. Code: https://github.com/yeridu/nmpou

**References**

Karlson KB, Holm A, Breen R. Comparing regression coefficients between same-sample nested models using logit and probit: a new method. Sociological Methodology. 2012;42:286-313. doi:10.1177/0081175012444861.

Breen R, Karlson KB, Holm A. Total, direct, and indirect effects in logit and probit models. Sociological Methods and Research. 2013;42(2):164-191. doi:10.1177/0049124113494572.

Allison PD. Comparing logit and probit coefficients across groups. Sociological Methods and Research. 1999;28(2):186-208. doi:10.1177/0049124199028002003.
