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

**2. The intuition: a direct payment or one routed through a middleman**

Picture a risk factor (say, bullying) as sending money to a single recipient, "opioid use." The money can travel by two routes:

- **Direct:** bullying hands the money straight to opioid use. For bullying, this is the social-pain-to-opioid pathway -- a channel that belongs to opioids alone.
- **Through a middleman:** bullying first pays into a general account, "uses many substances." Because opioids are one of the substances in that account, part of bullying's money reaches opioids only by passing through this middleman.

KHB is the forensic accountant. It opens the books and reports one number: of every dollar bullying sends to opioids, how many cents went direct and how many passed through the middleman.

- A truly opioid-specific risk factor pays almost entirely direct. Bullying did: only about 10 cents on the dollar (10.3%) traveled through the middleman.
- A broad risk factor routes a large share through the middleman. Sexual victimization did: about 25 cents on the dollar (25.2%) -- the signature of general rather than opioid-specific risk.

The whole method is just a careful way of doing this accounting so that the two routes are measured on the same scale. Section 3 explains why that turns out to be the hard part.

**3. Why you cannot just compare two simple models (the trap)**

The obvious approach would be to fit two logistic models -- one without polysubstance use (the "total" effect) and one with it (the "direct" effect) -- and read the shrinkage of the risk factor's coefficient as the indirect, mediated part. This works in ordinary linear regression. In logistic regression it does not, because of a property called noncollapsibility.

In plain words: in logistic regression, simply adding any variable changes the scale of all the coefficients (the model's built-in amount of unexplained variation is fixed, so adding a variable rescales everything). As a result, a coefficient can shrink when you add polysubstance use even if there is no real mediation at all -- a false impression of mediation. So the naive two-model comparison can mislead.

**4. The KHB fix in one idea**

KHB removes the trap by making the two models contain the **same number of variables**, so they sit on the **same scale** and can be compared cleanly. It does this with a trick: instead of dropping polysubstance use from one model, it **replaces** it with the part of polysubstance use that is not explained by the risk factor (its "residual").

Because that residual carries no information about the risk factor, putting it in the model lets the risk factor's coefficient absorb its full (total) effect; putting the real polysubstance measure in instead gives the direct effect. Both models have the same number of variables, so the difference between the two coefficients is a clean mediation estimate, free of the rescaling artifact. The next section shows exactly how that residual is built, because it is the part everything else depends on.

**5. How the residual is built (the heart of the fix)**

The fix in Section 4 rests entirely on one new ingredient: the residual of polysubstance use. Here is exactly how it is made and why it works.

**Step A -- predict polysubstance use from the risk factor.** Fit a survey-weighted **linear** regression with polysubstance use (coded 0/1) as the outcome and the risk factor -- plus the other risk factors and demographics -- as predictors. For each student this produces a predicted value: the amount of polysubstance use that the risk factor and the covariates can account for.

**Step B -- subtract.** The residual is the actual value minus the predicted value:

residual = (did the student actually use other substances? 0 or 1) minus (the value predicted from bullying and covariates)

A worked illustration with two bullied students:

| Student | Actually used other substances? | Predicted from bullying + covariates | Residual |
|---------|---------------------------------|--------------------------------------|----------|
| Ana | 1 (yes) | 0.70 | +0.30 |
| Beto | 0 (no) | 0.70 | -0.70 |

Ana uses more than her profile predicts (a positive residual); Beto uses less (a negative residual). The residual is "polysubstance use with the part explained by bullying stripped out" -- what remains after bullying's footprint is removed.

**Why this is the key.** By the mathematics of linear regression, this residual is **uncorrelated with the risk factor** (its correlation with bullying is exactly zero). That single property is what the method needs:

- Put the **real** polysubstance measure in the model and it shares information with bullying, so it absorbs the mediated part of bullying's effect; bullying's coefficient is then the **direct** effect.
- Put the **residual** in instead and it shares no information with bullying, so it absorbs nothing from bullying; bullying's coefficient returns to its **total** effect.

Both models still contain exactly one polysubstance term, so they sit on the same logistic scale. The gap between the two coefficients is therefore clean mediation, not the rescaling artifact of Section 3.

(The linear regression in Step A is deliberate. Only ordinary least squares guarantees a residual that is exactly uncorrelated with the predictors, which is the property the whole method rests on. This holds even though polysubstance use is a yes/no variable.)

**6. The three steps (with the code and variable names)**

Using bullying (bu_1) as the example, the outcome is NMPOU (q49_1) and the mediator is lifetime polysubstance use (su_life, with a 0/1 numeric version su_life_num for the linear step).

Step 1 -- build the residual (the procedure explained in Section 5). Regress the mediator on the risk factor plus all the other risk factors and demographics, using survey-weighted linear regression, and save the residuals:

```r
stage1_bu <- svyglm(su_life_num ~ bu_1 + rb_1 + sc_1 + sv_1 + q2_1 + q3_1 + q4_1,
                    design = des_khb, family = gaussian())
resid_su_bu <- residuals(stage1_bu)
```

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

**7. A worked example**

Suppose bullying's total log-odds coefficient is 0.50 and its direct coefficient is 0.45. Then the indirect (mediated) part is 0.50 - 0.45 = 0.05, and the percent mediated is 0.05 / 0.50 times 100 = 10 percent. In this study bullying's mediated share was about 10.3 percent -- the rest of its association is direct.

**8. What the decomposition found**

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

**9. How to read the numbers honestly**

- This is an accounting decomposition, not causal mediation. With cross-sectional data we cannot claim that polysubstance use causally transmits the effect of bullying on opioids. KHB tells us what fraction of the risk factor's log-odds is statistically accounted for by its shared association with polysubstance use. A causal mediation claim would require temporal ordering (the mediator before the outcome) and no unmeasured confounders of the mediator-outcome relationship.
- The percent mediated is scale-free. Because both models share the same scale, the unexplained-variation term cancels out of the ratio, so the percent mediated is not distorted by the logistic-scale problem described in Section 3.

**10. How this relates to the original KHB method (what we kept, what we changed)**

The decomposition comes from Karlson, Holm, and Breen (2012), with the total-direct-indirect interpretation formalized by Breen, Karlson, and Holm (2013). It is worth being explicit about what we adopted and what we did differently.

*The shared problem.* In logistic and probit models, a coefficient is scaled by the model's unexplained variation, and adding a mediator rescales every coefficient even when nothing is truly mediated (Section 3). Comparing a risk factor's coefficient before and after adding the mediator therefore confounds real mediation with pure rescaling. This is the problem KHB was built to solve.

*What is similar to the original KHB method.*

- We use the central KHB device exactly as published: residualize the mediator against the risk factor, then compare the risk factor's coefficient with the real mediator (the direct effect) against its coefficient with the residual (the total effect).
- We read the difference between the two as the indirect (mediated) part and report it as a percent mediated, which is scale-free -- the main advantage KHB has over the naive two-model comparison.
- We keep KHB's requirement that both models contain the same predictors, so they share a single logistic scale.

*What is different in our study.*

- Survey design. The original method was developed for ordinary (often independent) samples. We implemented every step on the complex survey design of the YRBS: the stage-1 regression is survey-weighted (svyglm with the gaussian family), and the two stage-2 models are survey-weighted quasibinomial. The standard khb software package does not take this design directly, so we built the three steps by hand to respect the weights, strata, and clustering.
- Uncertainty. Instead of the package's model-based standard errors, our confidence intervals come from a design-based bootstrap that resamples primary sampling units within strata, which honors the survey design.
- One mediator, six risk factors. We run the same decomposition separately for each of the six adversities, always with a single mediator -- lifetime polysubstance use -- chosen because it is the concrete embodiment of the "general substance use" pathway that the frameworks predict.
- Interpretation. We treat the result as an accounting decomposition, not causal mediation (Section 9). The original papers present the algebra; whether it carries a causal meaning depends on assumptions (temporal order, no unmeasured confounding) that cross-sectional YRBS data cannot satisfy, so we are deliberately more cautious in wording than a causal-mediation application would be.
- Relation to the companion method. The reason the naive two-model comparison fails is the same logistic scale problem that Allison (1999) raised for comparing coefficients across groups in the stacked-outcome method. KHB's residualization is an elegant way around it that needs no estimate of any noise ratio, which is why the percent mediated is scale-free.

*In one sentence.* We use KHB's residualization and its scale-free percent-mediated exactly as published, but reimplement it by hand for a weighted, clustered survey sample with bootstrap intervals, and we report it as an accounting decomposition rather than a causal one.

**11. Where this lives in the code**

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
