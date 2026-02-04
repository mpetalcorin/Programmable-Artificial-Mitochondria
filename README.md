# Programmable-Artificial-Mitochondria
A manufacturing-grade DBTL workflow with QC and ML-guided candidate selection for engineering IMM-like ATP power modules.

Programmable artificial mitochondria, built as a manufacturable process, standardized build records, QC gates that expose leak and dispersion failures, and ML surrogates that predict ATP output, quantify uncertainty, and propose the next best designs for data-efficient iteration.

# IMM-Energy-Modules-DBTL
**A proof-of-concept, simulation-driven Design–Build–Test–Learn (DBTL) workflow for engineered inner-mitochondrial-membrane-like (IMM-like) energy modules, linking manufacturing build records to ATP output using interpretable ML, robustness maps, and surrogate-guided next-build proposals.**

## Overview
This repository contains a reproducible, notebook-first workflow that treats **bioenergetic membrane reconstitution** as a **manufacturable process**, not a one-off demonstration. The code simulates “build records” for IMM-like energy modules and shows how to:

- capture **manufacturing-style variation** (lot-to-lot variability),
- implement explicit **QC gates** (PASS, CONDITIONAL_PASS, FAIL),
- train an interpretable **surrogate model** to predict ATP output from build knobs,
- compute **uncertainty proxies** to support risk-aware decisions,
- generate **robustness maps** across leak and polydispersity,
- propose **next builds** using surrogate-guided candidate selection,
- export **publication-ready figures and tables**.

The design includes three operating modes:
- **LIGHT**: light-driven proton pumping coupled to ATP synthase,
- **NADH**: respiration-like NADH-driven proton pumping coupled to ATP synthase,
- **NONE**: control/background ATP mode.

Rate and stability magnitudes are benchmarked to peer-reviewed, PubMed-indexed literature cited in the manuscript and in the notebook outputs (Mitchell, 1961; Mitchell, 1966; Racker & Stoeckenius, 1974; Rigaud & Lévy, 2003; Schägger & Pfeiffer, 2000; Pfeiffer et al., 2003; Cogliati et al., 2016; Biner et al., 2020; Jarman et al., 2021).

### What gets generated
Running the notebook/script will generate:
- **Build records** (one JSON per build) in `outputs/build_records/`
- **Dataset CSV** in `outputs/data/`
- **Tables (CSV)** in `outputs/tables/`
- **Figures (PNG + PDF)** in `outputs/figures/`

## Quickstart

### 1) Create an environment
Choose one:

**Option A, conda**
```bash
conda env create -f environment.yml
conda activate imm-dbt
```
## Repository structure
```
IMM-Energy-Modules-DBTL/
├── notebooks/
│   └── imm_energy_modules_dbtl.ipynb
├── src/
│   └── imm_energy_modules_dbtl.py
├── outputs/
│   ├── figures/
│   ├── tables/
│   ├── data/
│   └── build_records/
│   └── references.md
├── environment.yml
├── requirements.txt
└── README.md
└── Model Card.md
└── Datasheet.md
```
## What the workflow does (DBTL)
### Design
### Defines the controllable build knobs:
	•	membrane platform (lipid vs polymersome-like),
	•	lipid composition (cardiolipin, PE-mimic, cholesterol),
	•	detergent identity and concentration,
	•	assembly method and process parameters,
	•	operating mode (LIGHT, NADH, NONE).
### Build
Generates one manufacturing-style build record per build, capturing:
	•	inputs (knobs),
	•	process conditions,
	•	structural proxies (size, PDI, aggregation),
	•	leak proxies,
	•	functional outputs (pmf proxy, ATP rates/yields, on/off ratio, stability proxy).
### Test
Applies rule-based QC gates and assigns:
	•	PASS,
	•	CONDITIONAL_PASS,
	•	FAIL.
### Learn
Trains an interpretable random forest surrogate to predict ATP output and computes:
	•	held-out test performance (R², MAE),
	•	uncertainty proxy (ensemble std across trees),
	•	permutation importance,
	•	partial dependence curves,
	•	robustness maps (PASS rate in leak × PDI bins).
### Propose
Samples a candidate library in manufacturable ranges, scores candidates, and returns:
	•	top proposals (table),
	•	ATP distribution (histogram),
	•	trade-off plots (ATP vs leak),
	•	lipid-space maps (composition vs predicted ATP).
## Key outputs
### Figures
Typical figures include:
	•	ATP rate by mode (boxplots),
	•	leak vs ATP scatter,
	•	lipid composition maps (CL vs DOPE),
	•	robustness heatmaps (PASS rate),
	•	surrogate performance (predicted vs observed),
	•	uncertainty diagnostics (residuals vs uncertainty),
	•	interpretability (permutation importance, partial dependence),
	•	candidate selection plots.
### Tables
Typical tables include:
	•	summary statistics by mode,
	•	top observed high-output builds,
	•	model performance metrics,
	•	test predictions with uncertainty,
	•	top permutation importance features,
	•	robustness bin summaries,
	•	top proposed next builds.
### How to adapt to real experimental data
To use this workflow with real experiments:
	1.	Keep the build record schema, but replace simulated readouts with measured:
	•	ATP time course,
	•	membrane potential / Δψ,
	•	ΔpH probes,
	•	leak measures,
	•	size/PDI (DLS/NTA),
	•	insertion/orientation assays.
	2.	Use the same QC gates (tune thresholds to your assay).
	3.	Retrain the surrogate and re-run candidate proposal.
## Citing
If you use this repository, please cite the accompanying manuscript.
## Contact
Mark I.R. Petalcorin
London, UK
GitHub: https://github.com/mpetalcorin

# Model Card: IMM-Energy-Modules-DBTL Surrogate (Random Forest Regressor)

## Model details
**Model name:** IMM-Energy-Modules-DBTL Surrogate  
**Model type:** Random forest regression surrogate model  
**Primary task:** Predict **ATP production rate** (µM/min) for IMM-like engineered energy-module builds from build-record features.  
**Version:** v0.1 (proof-of-concept)  
**Implementation:** scikit-learn `RandomForestRegressor` wrapped in a `Pipeline` with `ColumnTransformer` + `OneHotEncoder`.  
**Developed by:** Mark Ihrwell R. Petalcorin  
**License:** MIT or Apache-2.0

## Intended use
**Primary intended use**
- Decision-support in a **Design–Build–Test–Learn (DBTL)** workflow for IMM-like energy modules.
- Rank candidate builds for follow-up, prioritize robust operating regimes, and identify actionable knobs (e.g., leak, detergent window, lipid envelope).

**Intended users**
- Researchers engineering reconstituted bioenergetic membrane systems (proteoliposomes, polymersomes, supported patches).
- Teams implementing closed-loop optimization for biochemical manufacturing workflows.

**Out-of-scope uses**
- Clinical decision making.
- Predicting in vivo mitochondrial bioenergetics or disease phenotypes.
- Claims of mechanistic causality without experimental validation.

## Model inputs and outputs
**Inputs**
A single row from a build record (tabular features), including:

- **Categorical features (one-hot encoded):**
  - `mode` (LIGHT, NADH, NONE)
  - `membrane_material` (lipid, polymersome)
  - `assembly_method`
  - `detergent_removal_method`
  - `detergent_type`
  - `visible_aggregation`

- **Numeric features:**
  - Buffer/process: `buffer_pH`, `ionic_strength_mM`, `temp_C`, `mixing_rpm`, `assembly_duration_min`
  - Composition: `lipid_CL_frac`, `lipid_DOPE_frac`, `lipid_DOPC_frac`, `cholesterol_frac`
  - Protein/process ratios: `atpsyn_ratio_value`, `detergent_conc_mM`
  - Physical proxies: `size_mean_nm`, `size_pdi`, `encapsulation_leak_rate_pct_per_h`
  - Coupling/orientation proxies: `protein_incorporation_fraction`, `orientation_proxy_score`
  - Functional proxies: `pmf_proxy_AU`, `on_off_ratio`
  - Mode-specific knobs (0 when not used): illumination and NADH/stoichiometry variables

**Output**
- `atp_rate_uM_per_min` (continuous regression): predicted ATP production rate (µM/min).
- Optional uncertainty proxy: standard deviation across trees, `y_pred_std` (µM/min).

## Training data
**Data type:** Simulated dataset generated by the repository’s code.  
**Data format:** Tabular “build records” + functional outcomes, exported as CSV and JSON per build.  
**Key assumption:** The simulation encodes **physics-inspired constraints** (chemiosmotic coupling), with magnitudes benchmarked to **PubMed-indexed** reconstitution literature on ATP generation in vesicles/polymersomes and NADH-fueled minimal respiration assemblies.

**Important note**
- The model is trained on **synthetic data** and is intended as a workflow demonstration. It is not validated on real wet-lab datasets.

## Evaluation
**Evaluation split**
- Held-out test split (train/test) performed within the notebook/script.

**Metrics**
- **R²** on held-out test set.
- **MAE** (µM/min) on held-out test set.

**Uncertainty behavior**
- Tree-ensemble prediction variance is used as a pragmatic uncertainty proxy.
- Residuals increasing with uncertainty is treated as a desirable decision-support signal for DBTL candidate selection.

## Interpretability and explainability
The workflow produces:
- **Permutation importance** to estimate which features drive predictive performance.
- **Partial dependence plots** for key continuous knobs to reveal nonlinear “windows” and interaction-like effects.

These are used to generate actionable hypotheses for subsequent build cycles, consistent with data-driven experimental iteration.

## Limitations
- **Synthetic-only training:** No direct wet-lab validation, results depend on simulation assumptions.
- **Proxy variables:** `pmf_proxy_AU`, `orientation_proxy_score`, and leak proxies are placeholders for real assays.
- **Not mechanistic:** The model is predictive, not a mechanistic electrochemical model of Δψ/ΔpH dynamics.
- **Domain shift risk:** If applied to real data without re-training and calibration, predictions may be unreliable.

## Ethical considerations
- The model does not process personal data.
- Outputs should not be used to justify biomedical or clinical claims.
- Any real-world deployment should include transparent documentation of data provenance, assay constraints, and uncertainty handling.

## Recommendations for real-world use
To transition from proof-of-concept to experimental utility:
1. Replace simulated readouts with measured assays (ATP time course, Δψ/ΔpH, leak, size/PDI).
2. Preserve the build-record schema and QC gates for traceability (Rigaud & Lévy, 2003).
3. Retrain the surrogate on real build data and validate with strict held-out lots/batches.
4. Consider calibration of uncertainty (e.g., conformal prediction) and active-learning selection strategies.

## References (conceptual anchors)
Chemiosmosis and coupling:
- Mitchell, P. (1961). Coupling of phosphorylation to electron and hydrogen transfer by a chemiosmotic type of mechanism. *Nature*. https://doi.org/10.1038/191144a0
- Mitchell, P. (1966). Chemiosmotic coupling in oxidative and photosynthetic phosphorylation. *Biological Reviews*. https://doi.org/10.1111/j.1469-185X.1966.tb01501.x

Reconstitution methods and failure modes:
- Rigaud, J.-L., & Lévy, D. (2003). Reconstitution of membrane proteins into liposomes. *Methods in Enzymology, 372*, 65–86. https://doi.org/10.1016/S0076-6879(03)72004-7
- Racker, E., & Stoeckenius, W. (1974). Reconstitution of purple membrane vesicles catalyzing light-driven proton uptake and ATP formation. *Journal of Biological Chemistry, 249*(2), 662–663. https://pubmed.ncbi.nlm.nih.gov/4272126/

IMM organization and lipid rationale:
- Cogliati, S., Enriquez, J. A., & Scorrano, L. (2016). Mitochondrial cristae: Where beauty meets functionality. *Trends in Biochemical Sciences, 41*(3), 261–273. https://doi.org/10.1016/j.tibs.2016.01.001
- Schägger, H., & Pfeiffer, K. (2000). Supercomplexes in the respiratory chains of yeast and mammalian mitochondria. *The EMBO Journal, 19*(8), 1777–1783. https://doi.org/10.1093/emboj/19.8.1777
- Pfeiffer, K., Gohil, V., Stuart, R. A., Hunte, C., Brandt, U., Greenberg, M. L., & Schägger, H. (2003). Cardiolipin stabilizes respiratory chain supercomplexes. *Journal of Biological Chemistry, 278*(52), 52873–52880. https://doi.org/10.1074/jbc.M308366200

Respiration-like minimal systems precedent:
- Biner, O., Fedor, J. G., Yin, Z., Hirst, J., & Hatzakis, N. S. (2020). Bottom-up construction of a minimal system for cellular respiration and energy regeneration. *Proceedings of the National Academy of Sciences of the United States of America*. https://pmc.ncbi.nlm.nih.gov/articles/PMC7611821/

# Datasheet for Dataset: IMM-Energy-Modules-DBTL (Simulated Build Records)

## 1. Dataset name
**IMM-Energy-Modules-DBTL: Simulated Build Records for IMM-like Energy Modules**

## 2. Dataset summary
This dataset contains **manufacturing-style build records** for simulated inner-mitochondrial-membrane-like (IMM-like) engineered energy modules. Each row represents one “build” (a lot/run) with:

- **Controllable inputs** (lipid composition, detergent conditions, assembly method, process parameters, operating mode),
- **Structural proxies** (size, polydispersity index, aggregation),
- **Barrier-function proxies** (leak rate),
- **Functional readouts** (pmf proxy, ATP rate, ATP yield, on/off ratios, stability proxy),
- **QC flags and a final QC label** (PASS / CONDITIONAL_PASS / FAIL).

The dataset was generated to demonstrate a **Design–Build–Test–Learn (DBTL)** workflow for engineered bioenergetic assemblies and to support surrogate modeling, interpretability, robustness mapping, and candidate proposal.

## 3. Motivation, purpose, and intended use
**Why was this dataset created?**
- To provide a proof-of-concept dataset for treating bioenergetic reconstitution as a **manufacturable process** with explicit build records, QC gates, and data-driven iteration.

**Intended uses**
- Training and testing surrogate ML models that predict ATP output from build-record features.
- Demonstrating interpretability tools (permutation importance, partial dependence).
- Robustness analysis across leak and dispersity.
- Proposing next experiments via surrogate-guided candidate selection.

**Out-of-scope uses**
- Making claims about in vivo mitochondrial physiology, disease, or clinical outcomes.
- Mechanistic inference beyond what the simulation encodes.
- Any decision-making that requires experimentally validated values.

## 4. Dataset composition
**What are the instances?**
- Each instance is one simulated **build** (one engineered membrane assembly attempt).

**How many instances?**
- Determined by the simulation run parameter (e.g., `n_builds=650` or `n_builds=800`).

**What does each instance contain?**
Typical feature groups include:
- **Identifiers and provenance:** `build_id`, schema and SOP versions, timestamp.
- **Operating mode:** `mode` ∈ {LIGHT, NADH, NONE}.
- **Membrane platform:** `membrane_material` (lipid, polymersome) and optional platform fields.
- **Composition:** lipid fractions (CL, DOPE proxy, DOPC proxy) and cholesterol.
- **Assembly choices:** detergent type/concentration, removal method, assembly method.
- **Process parameters:** temperature, mixing, duration, extrusion settings.
- **Structural proxies:** size mean, PDI, aggregation.
- **Barrier proxy:** leak rate (%/h).
- **Functional readouts:** pmf proxy, ATP rate (µM/min), ATP yield at 30 min (µM), on/off ratio.
- **QC flags:** background OK, size OK, leak OK, on/off OK, etc.
- **QC label:** `qc_status` ∈ {PASS, CONDITIONAL_PASS, FAIL}.

**Is there a target label?**
- Yes. The primary regression target is typically:
  - `atp_rate_uM_per_min`
- Secondary outcomes include:
  - `atp_yield_30min_uM`, `on_off_ratio`, `pmf_proxy_AU`
- QC label is available:
  - `qc_status`

## 5. Data collection process
**How was the data acquired?**
- The dataset is **simulated** by code in this repository.
- Values are generated via physics-inspired relationships encoding:
  - chemiosmotic coupling constraints (pmf must persist),
  - coupling/orientation/insertion effects,
  - leak penalties,
  - process-induced variability (e.g., aggregation, polydispersity).
- Magnitudes are tuned to match plausible orders of magnitude reported in PubMed-indexed reconstitution studies and chemiosmotic principles (Mitchell, 1961; Mitchell, 1966; Racker & Stoeckenius, 1974; Rigaud & Lévy, 2003; Biner et al., 2020).

**Who collected the data?**
- Generated algorithmically by the simulation code.

**Over what time frame?**
- Generated at runtime; timestamps reflect run time.

## 6. Preprocessing, cleaning, and labeling
**Preprocessing**
- Minimal preprocessing is required for most analyses.
- Categorical fields are one-hot encoded at model time.

**Labeling**
- The QC label `qc_status` is generated by **rule-based QC gates** that evaluate:
  - background ATP threshold,
  - leak threshold,
  - size/PDI window,
  - on/off ratio threshold,
  - controls behavior (assumed OK in the simulation).

## 7. Uses
**Primary uses**
- Demonstrate DBTL workflow structure for engineered bioenergetics.
- Train interpretable surrogate models for ATP output prediction.
- Explore robustness envelopes and process sensitivity.

**Potential secondary uses**
- Teaching dataset for ML interpretability on realistic mixed-type lab data.
- Prototyping active-learning or Bayesian optimization pipelines (with caution).

## 8. Distribution
**Where is the dataset stored?**
- Generated into:
  - `outputs/data/saam_builds.csv` (or similarly named CSV)
  - `outputs/build_records/*.json` (one JSON per build)

**How is it accessed?**
- By running the notebook/script to generate the dataset locally.

## 9. Maintenance
**Who maintains the dataset?**
- Repository owner/author.

**Will it be updated?**
- Yes, as the simulation improves or as real experimental datasets replace proxies.

## 10. Legal and ethical considerations
- This dataset contains **no personal data**.
- Because it is synthetic, it is not constrained by patient privacy considerations.
- Any real-world or experimental use should clearly disclose that this is simulated and validate against real measurements.

## 11. Known limitations
- **Synthetic-only:** Not experimentally validated.
- **Proxy variables:** pmf, orientation, and leak are placeholders for real assays.
- **Simplified physics:** No explicit Δψ/ΔpH electrochemical modeling, buffering, ionic flux, or mechanistic respiratory kinetics.
- **Domain shift risk:** Models trained on this dataset will not generalize to real experiments without retraining on measured build data.

## 12. Recommended citation (dataset)
If you publish work using this dataset, cite the associated manuscript in `manuscript/` and the conceptual anchors:

- Mitchell, P. (1961). Coupling of phosphorylation to electron and hydrogen transfer by a chemiosmotic type of mechanism. *Nature*. https://doi.org/10.1038/191144a0  
- Mitchell, P. (1966). Chemiosmotic coupling in oxidative and photosynthetic phosphorylation. *Biological Reviews*. https://doi.org/10.1111/j.1469-185X.1966.tb01501.x  
- Racker, E., & Stoeckenius, W. (1974). Reconstitution of purple membrane vesicles catalyzing light-driven proton uptake and ATP formation. *Journal of Biological Chemistry, 249*(2), 662–663. https://pubmed.ncbi.nlm.nih.gov/4272126/  
- Rigaud, J.-L., & Lévy, D. (2003). Reconstitution of membrane proteins into liposomes. *Methods in Enzymology, 372*, 65–86. https://doi.org/10.1016/S0076-6879(03)72004-7  
- Biner, O., Fedor, J. G., Yin, Z., Hirst, J., & Hatzakis, N. S. (2020). Bottom-up construction of a minimal system for cellular respiration and energy regeneration. *Proceedings of the National Academy of Sciences of the United States of America*. https://pmc.ncbi.nlm.nih.gov/articles/PMC7611821/
