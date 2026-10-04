# CausalMedia-GH

**CausalForestDML for Heterogeneous Estimation of Structured Content Engagement Effects on Academic Performance: Evaluated Against a Known-Ground-Truth Benchmark, with a Ghana Data-Readiness Blueprint**

*[Repository anonymised for double-blind review. Author identities, institutional affiliation, and supervision are withheld and will be restored on acceptance.]*

This repository accompanies a manuscript under review, developing and evaluating a `CausalForestDML`-based estimator of *heterogeneous* content-engagement effects on learning outcomes — on a public dataset that genuinely contains the required treatment, confounder and outcome measures — then translating the method into a teacher-facing decision-support application and a data-readiness blueprint for resource-constrained education systems.

> Motivated, better-resourced students tend both to engage more with course content *and* to perform better independently of that engagement — a classic selection-effect confound. This repository uses `CausalForestDML` (Microsoft EconML) to adjust for measured confounders and estimate each student's individual conditional average treatment effect (CATE) rather than a single population average.

---

## Table of Contents

- [Why This Motivates a Data-Readiness Blueprint](#why-this-motivates-a-data-readiness-blueprint)
- [Key Findings](#key-findings)
- [What This Study Does Not Claim](#what-this-study-does-not-claim)
- [Repository Structure](#repository-structure)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Reproduction](#reproduction)
- [Data Sources](#data-sources)
- [Methodology Summary](#methodology-summary)
- [Web Application](#web-application)
- [Limitations](#limitations)
- [Open Items](#open-items)
- [Ethical Compliance](#ethical-compliance)
- [License](#license)

---

## Why This Motivates a Data-Readiness Blueprint

Many education systems reliably capture institutional-level census data — enrolment, staffing, facilities — but not individual-level learning-process data: the combination of a directly measured engagement indicator, high-resolution behavioural confounders and a linked continuous outcome that heterogeneous causal-effect estimation requires. Emerging technology-access programmes in various settings are beginning to create the technical capacity to log individual interaction data, but analytics pipelines and data standards to capture, structure or act on it often lag behind.

This project makes **no quantitative claim about any population outside the dataset analysed.** It evaluates a causal-forest estimator on a public dataset that genuinely contains the required measures, so that a working method and a working decision-support system exist ahead of infrastructure that can generate individual-level engagement data elsewhere.

---

## Key Findings

| Metric | Value |
|---|---|
| Population ATE, primary corpus (n = 17,529) | **0.004786482048864249** per content-access click, 95% CI [0.002576, 0.006997] |
| Interpretable effect size | **3.7417 marks** of *reduced decline* per 1 SD of engagement (SD = 781.7294) |
| Heterogeneity is real | Observed CATE variance **10.4×** a constant-effect null mean, 41.3σ, p at the floor |
| Refutation | **4 of 4** DoWhy tests pass at 100 simulations, deviations under 2% |
| Seed stability | 0.004788 ± 0.000057 across five seeds — 1.3% of the analytic interval width |
| Primary heterogeneity finding | prior education — ε² = **0.0711**, compensatory gradient of **1.23×** within module |
| Sensitivity | An unmeasured confounder at **~40%** of course-identity strength drives the estimate to zero |
| Robustness subsample | 0.0646, 95% CI [−0.0283, 0.1576] — directionally consistent, **40.6%** of the primary per-SD magnitude, nested |

**The effect is a reduction in the rate of decline, not an improvement in marks.** Mean `performance_gain` across the corpus is −6.02, because assessments become more demanding as a course progresses.

Practical magnitude varies enormously by course, since a per-click effect means nothing without the click count it multiplies — reported in full in the accompanying manuscript.

---

## What This Study Does Not Claim

- **Not a validated estimator.** On the ground-truth benchmark, `CausalForestDML` is beaten by its own linear ablation on individual-level accuracy (PEHE) at all eight leaf configurations tested. The framework is *evaluated against* a known-ground-truth benchmark; it is not validated by it.
- **Not multimedia-specific.** The primary treatment is structured-content access, undifferentiated by media type.
- **Not robust — conditionally robust.** An unmeasured confounder at roughly 40% of course-identity strength would remove the effect.
- **No teacher-comprehension result.** The evaluation instrument exists; no sessions have run and no claim rests on it.
- **No population-level claim beyond the dataset analysed.**

---

## Repository Structure

```
CausalMedia-GH/
├── analysis-1/                 # Corpus construction, verified loader
├── analysis-2/                 # Ground-truth benchmark, leaf sweep, diagnostic
├── analysis-3/                 # Causal DAG definition
├── analysis-4/                 # CausalForestDML estimation
│   └── stability-supplement/   # leaf strip, seeds, calibration, modality
├── analysis-5/                 # SHAP, refutation, heterogeneity, figures
├── docs/                       # Methods and Results documentation
├── requirements.txt
└── README.md
```

Seven notebooks, no standalone scripts.

---

## Tech Stack

**Pipeline** — Python 3.12, pandas, NumPy, SciPy · **EconML** 0.17.0 (`CausalForestDML`, `LinearDML`) · scikit-learn 1.6.1 · **DoWhy** 0.14 (refutation) · **SHAP** 0.52.0 via EconML's native `shap_values()` · NetworkX (DAG).

**Application** — React 19 + Vite + Tailwind · Express + Prisma + SQLite · Socket.io. Presents completed results; performs no inference of its own.

---

## Getting Started

```bash
pip install -r requirements.txt
jupyter notebook
```

Python 3.10+; notebooks were developed and executed in a cloud notebook environment under 3.12.

---

## Reproduction

Place the seven source CSVs in `raw/`. The loader verifies row counts and SHA-256 prefixes against the documented release and halts on mismatch.

Run in order:

1. `analysis-1/` — corpus construction; expect n = 17,529, 40 confounder columns
2. `analysis-4/` — primary estimation; expect ATE `0.004786482048864249`
3. `analysis-4/stability-supplement/` — leaf strip, five seeds, calibration, modality
4. `analysis-5/` (refutation notebook) — four refuters at 100 simulations
5. `analysis-2/` — ground-truth benchmark and leaf sweep

The canonical model artefact is verified on load against the ATE constant to 1e-6.

---

## Data Sources

| Dataset | Type | Size | Role |
|---|---|---|---|
| A public learning-analytics dataset, CC-BY 4.0 | Public | 32,593 → 17,529 primary (6 of 7 courses); 2,031 subsample | Primary analytical corpus |
| A semi-synthetic benchmark with documented ground truth | Research benchmark | 672 × 25 × 100 realizations | Estimator evaluation only — never merged with the primary corpus |

Full citation details for both datasets are given in the accompanying manuscript and are withheld here only where they would otherwise sit alongside identifying repository metadata.

**No synthetic data is used anywhere in the analytical pipeline.** The application's blueprint mode carries clearly labelled synthetic personas, illustrative only and never presented as findings.

**Treatment (two-tier).** Primary: aggregated structured-content access, all 22 course-presentations. Secondary: live-conferencing access, restricted to the three presentations where it occurs.

**Outcome.** Last valid assessment score minus first, raw scores rather than weighted grades, winsorised at the 1st/99th percentiles.

**Confounders (10).** Demographic, administrative and course-structural variables — one-hot encoded to 40 columns. Course identity was a late addition, discovered through a SHAP-driven investigation of an anomalous coefficient; adding it moved the ATE by 85%.

**Sampling frame.** Six exclusion steps, 46.2% total attrition. The sixth conditions on the treatment: students with zero recorded access are excluded, all from two courses. The estimand is a dose–response effect **among students who engaged at least once**, not a comparison of engaging versus not.

---

## Methodology Summary

1. **Baseline progression** — unadjusted correlation → OLS → Linear DML → `CausalForestDML`.
2. **Engineered operation** — REPLACE: DML's linear second stage replaced with a non-parametric honest causal forest.
3. **Ground-truth evaluation** — 100 benchmark realizations, eight leaf configurations.
4. **Refutation** — four DoWhy tests at 100 simulations.
5. **Calibration** — heterogeneity tested against a **constant-effect** null via parametric bootstrap.
6. **Sensitivity** — an unmeasured confounder simulated across strengths in SD units.
7. **Explainability** — SHAP via EconML's native method, attributions aggregated to source variable before interpretation.

---

## Web Application

A teacher-facing decision-support application presenting the completed study. It performs no inference — every number is precomputed and loaded from CSV. Two modes, never blended: *Evidence* serves real course data with genuine estimator output; *Blueprint* serves a small, clearly labelled synthetic cohort behind a persistent SIMULATED banner. **Source repository withheld for review; details in the accompanying manuscript.**

---

## Limitations

- **Measurement ceiling.** The treatment resolves access and nothing else — not duration, attention, comprehension, sequence, media type or assessment alignment.
- **Selection on a collider.** The corpus conditions on completion, which engagement plausibly causes.
- **Selection on the treatment.** Students with zero recorded access are absent; the conditioning falls on two of six courses only.
- **Conditional robustness.** An unmeasured confounder at ~40% of course-identity strength drives the estimate to zero.
- **Two inference procedures disagree.** The analytic interval is 3.02× the empirical resampling one.
- **Benchmark outcome.** The estimator underperforms its linear ablation on individual-level accuracy on the only data where accuracy is measurable.
- **Corpus composition.** Advantaged students are over-represented on every marker examined.

---

## Open Items

One course's estimate is under active diagnosis and no conclusion should be drawn from it in isolation — see the accompanying manuscript for the full sample-size analysis.

Also outstanding: dependency pinning completeness; application source integration; teacher-comprehension sessions (instrument ready, not yet run).

---

## Ethical Compliance

The primary dataset is public, pre-anonymised secondary data under CC-BY 4.0, with no identifiable personal data and no primary human-subjects contact — no additional approval was required for its analysis.

The teacher usability evaluation is separate primary human-subjects research. **Institutional ethical clearance has been granted.** No data has been collected. Stimuli are synthetic profiles derived from the corpus distributions, disclosed to participants as illustrative and not as records of real students.

---

## License

*[Not yet specified.]*

---

*Author identities, institutional affiliation, funding, and acknowledgements are withheld from this repository for double-blind review, and will be added on acceptance.*
