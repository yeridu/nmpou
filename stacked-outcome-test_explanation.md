---
output:
  word_document:
    reference_docx: "C:/Users/yerid/Documents/reference_doc.docx"
---

**Understanding the Stacked-Outcome Specificity Test**

**1. The question this test answers**

Many adversities are associated with adolescent non-medical prescription opioid use (NMPOU): bullying, suicidality, other drug use, a difficult home, and so on. But most of those same adversities are *also* associated with alcohol, marijuana, cigarettes, and almost every other substance. So a single question drives the whole analysis:

**Is a given adversity linked specifically to opioids, or is it linked to substance use in general?**

The stacked-outcome test is what lets us answer that question for each adversity.

**2. The intuition: a detective and five suspects**

Think of yourself as a detective.

- The adversity (say, bullying) is a clue.
- The five substances are five suspects: opioids, alcohol, marijuana, cigarettes, cocaine.

There are two kinds of clues:

- A general clue points to everyone (useless for naming a culprit). Example: "the suspect breathes air" -- all five do.
- A specific clue points to one suspect. Example: "the suspect has a scar on the left hand" -- only one does.

The stacked-outcome test decides which kind of clue each adversity is. In this study, bullying behaved like a specific clue (it pointed only to opioids), while polysubstance use behaved like a general clue (it pointed to all five).

**3. Why not just run five separate models?**

A natural first idea is to fit one model per substance and eyeball the results: "bullying is significant for opioids but not for alcohol." We do report those per-substance numbers (they appear in Figure 2 of the paper). But that is not a fair, formal comparison. "Significant for opioids and not for alcohol" does not statistically prove that the bullying effect is *larger* for opioids than for alcohol; the two could be statistically indistinguishable.

To compare formally, all substances must be in one model, with a single test of the difference. That is exactly what stacking makes possible.

**4. What "stacked" means (the key idea)**

Normally, each student is one row of data. Here, we copy each student into five rows, one per substance, and pile (stack) them into one long table.

In every one of a student's five rows:

- the adversity values are the same (a student's bullying does not change),
- Y answers "did this student ever use THIS row's substance?" (yes/no),
- outcome_type is the label saying which substance the row is about.

*A worked example.* Take a student, "Ana," who was bullied and who, in her life, used alcohol and marijuana but not opioids, cigarettes, or cocaine. Ana becomes five rows:

| Row | outcome_type | Y = used this substance? | bu_1 (bullied?) |
|-----|----------------|----------------------------|-------------------|
| 1 | Opioids (NMPOU) | No | Yes |
| 2 | Alcohol | Yes | Yes |
| 3 | Marijuana | Yes | Yes |
| 4 | Cigarettes | No | Yes |
| 5 | Cocaine | No | Yes |

Notice two things: Y changes from row to row because each row asks about a different substance, while bu_1 stays the same. The model then learns: among students who were bullied, in which substance-rows is Y = Yes more often than expected? If bullied students show extra "Yes" in the opioid rows but not the others, bullying is opioid-specific.

Two housekeeping details:

- Because each student now appears five times, the survey weights are divided by 5, so the population total is not inflated.
- Because a student's five rows are not independent (same person), the analysis clusters at the student level (id = psu + student), which corrects the uncertainty estimates.

In the script, each student's record is copied once per substance and the copies are stacked. Each copy keeps the same adversities but a different outcome Y and its matching leave-one-out polysubstance measure (NMPOU: Y = q49_1, su_loo_nmpou; Alcohol: q41_ever, su_loo_alc; Marijuana: q46_life, su_loo_mar; Cigarettes: q31_life, su_loo_cig; Cocaine: q50_1, su_loo_coc):

```r
# the opioid (NMPOU) copy shown; r2..r5 are the same for the other four substances
r1 <- transmute(sub_data, student_id, psu, stratum, weight,
                outcome_type = "NMPOU", Y = q49_1, su_loo = su_loo_nmpou,
                q2_1, q3_1, q4_1, rb_1, bu_1, sc_1, sv_1, hh_1, ms_1)
stacked5 <- bind_rows(r1, r2, r3, r4, r5)          # pile the 5 copies into one table
des_stacked5 <- svydesign(id = ~psu + student_id, strata = ~stratum,
                          weights = ~weight_stacked, data = stacked5, nest = TRUE)
```

(weight_stacked is the original survey weight divided by 5.)

**5. The variables in the model**

| In the code | What it means | Role |
|-------------|---------------|------|
| Y | Did the student use this row's substance? (yes/no) | Outcome |
| outcome_type | Which substance the row is about | The "group"; opioids = reference |
| bu_1 | Bullying victimization | Tested for specificity |
| sc_1 | Suicidality | Tested for specificity |
| sv_1 | Sexual victimization | Tested for specificity |
| rb_1 | Risk behavior | Tested for specificity |
| hh_1 | Household adversity | Tested for specificity |
| ms_1 | Community safety adversity | Tested for specificity |
| su_loo | Polysubstance use (leave-one-out; see Section 6) | Held equal across substances |
| q2_1, q3_1, q4_1 | Sex, grade, race/ethnicity | Controls, held equal across substances |
| each adversity by outcome_type | The difference in an adversity's effect across substances | This is where specificity is measured |

**6. The leave-one-out polysubstance measure (avoiding a trap)**

Polysubstance use (su_life) is defined as any use of cigarettes, alcohol, or marijuana (yes/no).

There is a trap. If the row's outcome is, say, alcohol, and the polysubstance measure also counts alcohol, the model would be predicting alcohol use partly with alcohol use -- a circular, near-tautological relationship. The fix is leave-one-out: in each row, the row's own substance is removed from the polysubstance measure and opioids are substituted in, so the measure always counts three substances but never the one being predicted.

In the script, for example, the alcohol row's measure su_loo_alc is coded Yes if the student used cigarettes (q31_life), marijuana (q46_life), or NMPOU (q49_1), and No otherwise. The NMPOU and cocaine rows use su_life (cigarettes q31_life + alcohol q41_ever + marijuana q46_life), which already excludes them.

| Row (outcome_type) | Polysubstance counts |
|----------------------|---------------------------------|
| Opioids (NMPOU) | cigarettes + alcohol + marijuana |
| Alcohol | cigarettes + marijuana + opioids (alcohol removed) |
| Marijuana | cigarettes + alcohol + opioids (marijuana removed) |
| Cigarettes | alcohol + marijuana + opioids (cigarettes removed) |
| Cocaine | cigarettes + alcohol + marijuana (cocaine was never in the trio) |

**7. The model (a logistic regression)**

The model is a survey-weighted logistic (logit) regression, not probit. (In the code, the quasibinomial family uses the logit link by default. Logit and probit are close cousins that share the same underlying idea; this study uses logit because its coefficients translate into odds ratios.)

Written as a formula, the model predicts Y from: outcome_type; the polysubstance measure su_loo; the six adversities (rb_1, bu_1, sc_1, sv_1, hh_1, ms_1); the demographics (q2_1, q3_1, q4_1); plus an interaction between each adversity and outcome_type (for example, bullying-by-outcome-type). It is fit with the survey package on the stacked survey design, using a quasibinomial (logit) family.

In the script, the exact code is:

```r
fit_stacked5 <- svyglm(
  Y ~ outcome_type + su_loo + rb_1 + bu_1 + sc_1 + sv_1 + hh_1 + ms_1 +
      q2_1 + q3_1 + q4_1 +
      bu_1:outcome_type + sc_1:outcome_type + sv_1:outcome_type +
      rb_1:outcome_type + hh_1:outcome_type + ms_1:outcome_type,
  design = des_stacked5, family = quasibinomial())
```

Reading the formula: everything before the interaction terms is a base effect (held the same across substances for su_loo and the demographics), and each "adversity:outcome_type" term is the per-substance difference described in Section 8.

Two design choices matter:

- outcome_type is included as a main effect, so each substance gets its own baseline (this accounts for the fact that some substances, like alcohol, are common while others, like cocaine, are rare).
- su_loo and the demographics carry no interaction with outcome_type, so their effect is held equal across substances. This common reference is what makes the other adversities' substance-specific differences identifiable and interpretable.

In words: predict the chance of using the row's substance from the adversities, allowing each adversity to act differently on each substance.

**8. How specificity is measured: the interaction terms**

Each adversity gets two kinds of number:

- a base effect: its effect in the reference substance, opioids (for example, the bullying main effect), call it B;
- a difference for each other substance (the bullying-by-outcome-type terms), call it D.

So the effect of bullying is assembled by addition:

- effect in opioids = B
- effect in alcohol = B + D(alcohol)
- effect in marijuana = B + D(marijuana)
- effect in cigarettes = B + D(cigarettes)
- effect in cocaine = B + D(cocaine)

Each D is, in plain terms, the bullying slope among that substance's rows minus the bullying slope among opioid rows -- that is, the difference. The regression estimates all the B's and D's together by finding the values that best fit every row at once (maximum likelihood).

In this study, bullying's D values for the other substances were negative and large enough to cancel B, pushing the effect to roughly zero for everything except opioids. That is the statistical signature of opioid specificity.

**9. The test is a separate step: the joint Wald test**

Fitting the model and testing a hypothesis are two different steps:

1. Fit the model. This produces the numbers (the B's and D's) and, for each, a margin of error.
2. Wald test. This asks a question about those numbers: are these differences really not zero, or could they be noise within the margin of error?

The specificity question is whether the D terms differ from zero. A joint Wald test evaluates several D's at once and returns a single result. In the script, the joint distress test is:

```r
regTermTest(fit_stacked5, ~ bu_1:outcome_type + sc_1:outcome_type, method = "Wald")
```

which returned F = 4.61, p = 0.0075. The small p-value (below 0.05) means the differences across substances are real, not chance: the distress adversities do not act the same on every substance. (Each adversity is also tested on its own in the same way, for example regTermTest with ~ bu_1:outcome_type for bullying alone.)

**10. What the test found**

- Bullying: associated with opioids (about +3.8 percentage points; adjusted odds ratio 1.48) and with none of the other four substances, so it is opioid-specific.
- Polysubstance use: large for every substance, so it is a general liability, not specific.
- Suicidality: opioid-specific in part (associated with opioids and marijuana, not alcohol).
- Sexual victimization, household adversity, community safety adversity: broad rather than opioid-specific.

A confirmatory three-substance model (opioids, alcohol, marijuana) showed the same pattern.

**11. How to read the numbers honestly**

- Primary effects are reported on the probability scale (average marginal effects, AMEs: the change in the probability of use when an adversity is present), because these are directly comparable across substances. A linear probability model produces the same pattern.
- Why the probability scale matters. A logistic coefficient is scaled by the unexplained "background noise" of its own model, and that noise can differ across substances; comparing raw logistic coefficients across groups can therefore be misleading (Allison, 1999; see Section 12). Probability-scale effects sidestep this problem, which is why they are the study's main estimand.
- Association, not causation. The data are cross-sectional, so these are associations that can guide screening and prevention, not proof that bullying causes opioid use.

**12. How this relates to Paul Allison's (1999) method**

The idea of comparing an effect across groups (here, across substances) without being fooled by each group's unexplained variation comes from Allison's 1999 paper, which is the original methodological source for this part of the analysis. It is worth being clear about what we borrowed and what we did differently.

*The shared problem.* A logistic (or probit) coefficient does not measure the effect by itself; it measures the effect divided by the amount of unexplained variation ("noise") in that group's model. Different groups can have different amounts of noise. So when you compare raw logistic coefficients across groups, a difference might reflect a real difference in the effect, or it might just reflect a difference in noise. The coefficients alone cannot tell the two apart. This is the core warning in Allison (1999).

*What is similar to Allison.*

- We treat the five substances as Allison's "groups" and recognize the same identification problem.
- We use Allison's key requirement: to make the comparison interpretable, some variables must be assumed to act the same in every group. In our model, polysubstance use and the demographics are held equal across substances (they carry no interaction with substance type). This is the equal-effect anchor that Allison's framework needs to identify the remaining differences.
- We follow Allison on the mechanics of stacking: the leave-one-out polysubstance measure (Section 6, to avoid circularity) and dividing the survey weights by the number of substances (Section 4, so the population is not multiplied) both follow his guidance.

*What is different from Allison.*

- Allison's full remedy is to fit a special model (a heteroscedastic logit) that actually estimates how much the noise differs between groups -- a scale ratio he sometimes writes as a single parameter -- and then adjusts the coefficients for it.
- This study does not estimate that noise ratio. Instead, it avoids the problem in a simpler, complementary way: the main results are reported on the probability scale (average marginal effects), which do not depend on the noise separately; they are confirmed by a linear probability model; and the KHB mediation shares are scale-free by construction.
- We also note that noise alone cannot explain our pattern. A single difference in noise would push all the substance comparisons in the same direction. Yet here bullying points toward opioids while sexual victimization points toward other substances -- opposite directions -- which a scale difference cannot create.

*In one sentence.* We adopt Allison's problem and his anchoring logic, but we solve the problem by moving to the probability scale rather than by estimating his noise-ratio correction; the latent-scale comparison Allison warns about is acknowledged as a limitation in the paper.

**13. Where this lives in the code**

All steps above are in the analysis script (nonmedicaluse_painmed_script_v4.Rmd):

- the leave-one-out polysubstance measures (su_loo_nmpou, su_loo_alc, su_loo_mar, su_loo_cig, su_loo_coc) are built in the composites chunk;
- the five-row stacked dataset (stacked5) is built by combining the five per-substance copies (r1 to r5) with bind_rows();
- the stacked model is fit_stacked5 (the confirmatory three-substance model is fit_stacked3);
- the specificity tests are the regTermTest() calls: the joint distress test, the full-model test, and the per-predictor tests (wald_bu, wald_sc, wald_sv, wald_rb, wald_hh, wald_ms).

Data: 2023 Youth Risk Behavior Survey (CDC), publicly available at https://www.cdc.gov/yrbs/. Code: https://github.com/yeridu/nmpou

**Reference**

Allison PD. Comparing logit and probit coefficients across groups. Sociological Methods and Research. 1999;28(2):186-208. doi:10.1177/0049124199028002003.
