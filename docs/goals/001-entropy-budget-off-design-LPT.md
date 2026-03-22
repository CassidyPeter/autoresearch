# Goal 001: Entropy Budget and Non-Equilibrium BL Physics in Off-Design LPT

## Definition

- **Target:** Quantify entropy generation budget across incidence sweep; identify the non-equilibrium BL region as the dominant variable loss contributor; validate RANS failure modes physically
- **Data:** DNS/iLES at i = 0°, +12°, −12° (SPLEEN blade, clean inflow, ArgoDG, Lucia HPC)
- **Status:** IN_PROGRESS
- **Paper target:** ASME Journal of Turbomachinery (Y3/Y4)
- **Novelty claim:** First entropy budget decomposition for off-design LPT; first non-equilibrium BL characterization in LPT context

---

## Central Claims to Establish

1. The non-equilibrium turbulent BL (post-reattachment) is the primary driver of the **incremental** loss with incidence — not the separation bubble itself
2. The divergence of turbulent production Pr and viscous dissipation D_visc (the non-equilibrium indicator from Wheeler 2018) scales with incidence in a characterisable way in LPT flows
3. RANS fails most severely precisely in the non-equilibrium BL region, and the failure correlates with Reynolds stress anisotropy
4. SPOD resolves distinct coherent structures at KH and TS frequencies inside the LSB, and their energy content shifts measurably with incidence

---

## Results Summary

| # | Hypothesis | Status | Key Finding |
|---|-----------|--------|------------|
| 0 | Baseline entropy budget at i=0° establishes reference decomposition | PLANNED | — |
| 1 | Entropy budget shifts at i=±12°: non-eq. BL contribution increases disproportionately | PLANNED | — |
| 2 | C_D divergence (Pr vs D_visc) is detectable at DNS resolution and scales with bubble length | PLANNED | — |
| 3 | SPOD identifies KH and TS modes; KH energy dominates at positive incidence, suppressed at negative | PLANNED | — |
| 4 | RANS Reynolds stress anisotropy error maps onto non-eq. BL region identified by entropy audit | PLANNED | — |

---

## Hypothesis 0: Baseline Entropy Budget at i = 0°

**Status:** PLANNED

**Reasoning:** Establish the reference entropy budget decomposition at design incidence before looking at the incidence effect. This is the foundation for all subsequent comparisons.

**Analysis tasks:**
1. Extract time-averaged entropy generation rate field `ṡ = Φ/T` from DNS
2. Define integration regions: (a) laminar SS BL, (b) LSB, (c) non-eq. turbulent SS BL, (d) eq. turbulent SS BL, (e) pressure side BL, (f) passage/freestream, (g) wake
3. Compute fractional loss contributions from each region
4. Compute C_D along the suction surface, decomposed into Pr and D_visc
5. Compare with Przytarski & Wheeler (2021) compressor results — is the LPT budget qualitatively similar?

**Expected result:** At i = 0°, LSB + transitional BL account for ~40-50% of total loss. Pr dominates C_D in the non-eq. region immediately post-reattachment; D_visc catches up as BL equilibrates.

**Success criteria:** Budget accounts for >95% of total downstream wake loss (mass-averaged total pressure). Individual region contributions are stable over time-averaging window.

**Config:** `configs/goals/001/h000-baseline-i0.yaml`

---

## Hypothesis 1: Incidence Effect on Entropy Budget

**Status:** PLANNED

**Reasoning:** If H0 establishes the reference, H1 asks: which regions change most as incidence moves to ±12°? The hypothesis is that the non-equilibrium BL post-reattachment amplifies disproportionately at positive incidence (longer bubble → longer non-eq. recovery) and may not exist at all at strongly negative incidence (no separation).

**Analysis tasks:**
1. Repeat H0 analysis at i = +12° and i = −12°
2. Plot entropy budget breakdown as a bar chart across three incidence angles
3. Quantify: absolute and fractional change in each region contribution
4. Measure: separation onset x_s, transition onset x_t, reattachment x_r from DNS skin friction
5. Correlate: change in non-eq. BL loss with (x_r - x_t) bubble length metric

**Expected result:** At i = +12°, non-eq. turbulent BL contribution increases by >50% relative to i = 0°. LSB contribution also increases but the post-reattachment BL recovery region dominates the incremental loss. At i = −12°, separation may be absent (attached-flow transition), so non-eq. contribution drops sharply.

**Success criteria:** Clear monotonic trend in non-eq. BL loss fraction with incidence. Correlation between bubble length and non-eq. loss contribution is physically interpretable.

**Config:** `configs/goals/001/h001-incidence-sweep.yaml`

---

## Hypothesis 2: Dissipation Coefficient Non-Equilibrium Indicator

**Status:** PLANNED

**Reasoning:** Wheeler (2018) showed for compressors that the **divergence** of Pr and D_visc along the blade surface — where Pr >> D_visc — is the signature of a non-equilibrium BL. The hypothesis is that this divergence is larger and extends further downstream in LPT at positive incidence, and that its integrated magnitude correlates with the excess loss above the equilibrium prediction.

**Analysis tasks:**
1. Compute C_D(s) along suction surface for each incidence case
2. Decompose into Pr(s) and D_visc(s) at each chordwise location
3. Plot Pr/D_visc ratio vs. chordwise position — identify the non-equilibrium zone where ratio > 1.5
4. Define non-eq. extent as the streamwise distance from reattachment to where Pr/D_visc → 1
5. Correlate non-eq. extent and peak Pr/D_visc with total profile loss

**Expected result:** Pr/D_visc ratio peaks near reattachment and decays downstream. The peak ratio and non-eq. extent both increase with positive incidence. This provides the physical mechanism for the excess loss — the BL takes longer to relax to equilibrium after the larger bubble.

**Success criteria:** Non-eq. extent and peak Pr/D_visc correlate with total loss with R² > 0.9 across the three DNS incidence cases. If experiments later provide C_D data at more incidence angles, this extends naturally.

**Config:** `configs/goals/001/h002-dissipation-coeff.yaml`

---

## Hypothesis 3: SPOD Coherent Structures in LSB

**Status:** PLANNED

**Reasoning:** The Genoa group (Lengani, Simoni, Dellacasagrande) has applied POD and wavelet methods to LSBs on flat plates but SPOD is not yet applied to LPT blades at off-design. SPOD with the entropy-production-weighted inner product will isolate which frequency ranges (KH roll-up, TS waves, shedding frequency) contribute most to entropy production.

**Analysis tasks:**
1. Extract time-resolved snapshots of entropy generation rate field over the LSB region
2. Apply standard SPOD (Towne et al. 2018 algorithm, Welch method) with kinetic energy kernel
3. Apply entropy-production-rate weighted SPOD: `<f,g>_s = ∫ ṡ_mean * f * g dΩ`
4. Identify dominant Strouhal numbers: `St = f * C_ax / U_in`
5. Compare KH roll-up frequency with linear stability theory estimate: `St_KH ≈ 0.032 * U_e / θ_sep`
6. Repeat at i = +12° and compare mode shapes and energy rankings

**Expected result:** At i = 0°, KH mode at St ≈ 0.3–0.5 (based on bubble length) dominates first SPOD mode. Entropy-weighted SPOD re-ranks modes compared to kinetic-energy SPOD — the most energetic velocity mode is not necessarily the most loss-producing. At i = +12°, KH mode energy increases; the larger separation provides more shear for KH amplification.

**Success criteria:** Clear spectral peaks identified in SPOD eigenvalue spectrum at physically expected frequencies. Entropy-weighted and kinetic-energy SPOD give observably different mode rankings for at least one case.

**Config:** `configs/goals/001/h003-spod-LSB.yaml`

---

## Hypothesis 4: RANS Failure Mode Mapping

**Status:** PLANNED

**Reasoning:** Once the entropy budget identifies where and how much loss is generated, the RANS failure mode can be precisely localised. The hypothesis is that RANS error is largest in the non-equilibrium turbulent BL region identified by H2, and that this correlates with the Reynolds stress anisotropy deviation from the Boussinesq approximation.

**Analysis tasks:**
1. Run RANS (OpenFOAM, RSM + Langtry-Menter) at same conditions as DNS
2. Extract RANS entropy generation rate field and compute budget decomposition (same regions as H0)
3. Compute: `ΔC_D = C_D_RANS - C_D_DNS` along suction surface
4. Map RANS failure locations onto the non-eq. BL zones identified in H2
5. Compute Reynolds stress anisotropy tensor `b_ij` from DNS; compute the same from RANS
6. Correlate `Δb_ij` (anisotropy error) with `ΔC_D` (C_D error)

**Expected result:** The largest ΔC_D values occur in the non-equilibrium BL zone, confirming the RANS fails where the Boussinesq approximation breaks down most severely. RANS over-predicts loss in the laminar BL and under-predicts in the non-eq. turbulent BL.

**Success criteria:** The spatial overlap between high `Δb_ij` and high `ΔC_D` is statistically significant. This provides a physical diagnostic for where RANS corrections (Weatheritt et al. 2017 evolutionary algorithm) should be applied.

**Config:** `configs/goals/001/h004-rans-failure.yaml`

---

## Paper Structure (Proposed)

1. Introduction — gap: no off-design LPT entropy budget, no non-eq. BL in LPT
2. Computational methodology — ArgoDG, mesh resolution criteria (Przytarski & Wheeler 2021 framework)
3. Baseline entropy budget (i = 0°) — H0
4. Incidence effect on entropy budget — H1
5. Non-equilibrium indicator: C_D divergence — H2
6. Coherent structures in the LSB: SPOD — H3
7. RANS failure localisation — H4
8. Towards a loss correlation — preliminary physical model connecting bubble length to non-eq. loss fraction
9. Conclusions
