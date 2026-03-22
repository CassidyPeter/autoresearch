# Goal 002: Wake-Induced Transition and Entropy Production in Off-Design LPT

## Definition

- **Target:** Quantify how incoming periodic wakes modify the entropy production budget at design and off-design incidence; characterise wake-LSB interaction as a function of incidence
- **Data:** DNS/iLES with incoming periodic wakes at i = 0°, ±12° (ArgoDG, Y3 simulations, DHIT precursor box)
- **Status:** PLANNED (awaiting Y3 DNS with injected turbulence / wake forcing)
- **Paper target:** ASME Turbo Expo 2026 (conference) → ASME Journal of Turbomachinery (journal extension)
- **Novelty claim:** First DNS study of wake-induced transition effects on entropy budgets in off-design LPT; first characterisation of wake-incidence interaction on LSB suppression/modification

---

## Central Claims to Establish

1. Incoming wakes **suppress** the laminar separation bubble at design incidence via bypass transition; this shifts the entropy budget from LSB-dominated to wake-dominated production
2. At positive incidence, the bubble is larger and wakes only **partially suppress** it — the non-eq. turbulent BL post-reattachment persists, resulting in higher total loss than at design incidence despite wake-induced reduction of the LSB
3. At negative incidence, where attached-flow transition prevails, wake impingement triggers **bypass transition earlier** — the entropy production shifts upstream relative to the clean-inflow case
4. SPOD of the wake-LSB interaction region reveals distinct modes: wake-induced streaks (Klebanoff modes) separate from KH roll-up modes — their relative energy rankings change with incidence

---

## Results Summary

| # | Hypothesis | Status | Key Finding |
|---|-----------|--------|------------|
| 0 | Differential entropy budget: wakes vs. clean inflow at i = 0° | PLANNED | — |
| 1 | Incidence modifies wake-bubble interaction: partial suppression at i = +12° | PLANNED | — |
| 2 | SPOD of wake-LSB interaction distinguishes streak modes from KH modes | PLANNED | — |
| 3 | Intermittency analysis: wake passage triggers turbulent spots, quantify production rate | PLANNED | — |

---

## Hypothesis 0: Differential Entropy Budget at i = 0° with Wakes

**Status:** PLANNED

**Reasoning:** The first question is simply: how does the entropy budget decomposition from Goal 001 (clean inflow) change when wakes are introduced at design incidence? This has never been done for LPT with wakes.

**Analysis tasks:**
1. Run DNS at i = 0° with incoming wakes (periodic bar wake or DHIT precursor, Strouhal number St_w ≈ 0.5–1.0 typical of engine conditions)
2. Compute time-averaged entropy generation rate — same regional decomposition as Goal 001 H0
3. Compute **phase-averaged** entropy budget: lock to wake passing period to see evolution over wake cycle
4. Quantify: Δ(loss_LSB), Δ(loss_non-eq. BL), Δ(loss_wake) relative to clean-inflow case
5. Check: does total loss increase or decrease with wakes? (Expected: marginally decreasing for moderate wake strength due to LSB suppression, but depends on wake width and convective velocity ratio)

**Expected result:** Wake suppresses or shortens the LSB — the LSB contribution to total loss decreases. However, the transition moves upstream and the turbulent BL is longer — the non-eq. BL contribution increases. Net loss effect depends on the tradeoff. The wake itself adds ~10-15% additional freestream loss (from Przytarski & Wheeler: freestream doubled from 15% to 30% at Tu = 4%→6%).

**Success criteria:** Phase-averaged entropy production shows clear wake-passage modulation. Loss budget decomposition is self-consistent and accounts for >95% of downstream total pressure loss.

**Config:** `configs/goals/002/h000-wake-i0-diff-budget.yaml`

---

## Hypothesis 1: Incidence Effect on Wake-Bubble Interaction

**Status:** PLANNED

**Reasoning:** The key question for novelty: does the wake-induced suppression of the LSB remain effective at off-design incidence? At large positive incidence, the bubble is much larger and more robust — does a wake of the same strength still suppress it? This has direct engineering relevance for contrail-avoidance flight at off-design conditions.

**Analysis tasks:**
1. Run DNS with wakes at i = +12° and i = −12°
2. Compute differential entropy budgets vs. clean-inflow cases (Goal 001 H1)
3. Compute "wake effectiveness" metric: `η_w = (loss_clean - loss_wake) / loss_clean` for the LSB region
4. Compare η_w across all three incidence angles
5. Identify: at which incidence does wake suppression of the LSB break down?
6. Measure: phase-averaged Cp distribution during wake passage — does the bubble disappear entirely or just shorten?

**Expected result:** At i = 0°, η_w > 0 (wake reduces LSB loss). At i = +12°, η_w is smaller or even negative (wake cannot suppress the enlarged bubble, total loss may increase). At i = −12°, η_w addresses bypass transition timing rather than bubble suppression — different physics applies.

**Success criteria:** Clear trend in η_w with incidence. Physical interpretation connects to bubble size (from H1 of Goal 001) and wake momentum thickness relative to bubble displacement thickness.

**Config:** `configs/goals/002/h001-incidence-wake-effectiveness.yaml`

---

## Hypothesis 2: SPOD of Wake-LSB Interaction

**Status:** PLANNED

**Reasoning:** When wakes interact with an LSB, two competing mechanisms are present: (1) wake-induced Klebanoff streaks (bypass transition mechanism, 3D) and (2) KH roll-up in the separated shear layer (2D). SPOD should separate these because they occur at different frequencies. This is the first SPOD analysis of wake-LSB interaction in LPT at off-design conditions.

**Analysis tasks:**
1. Extract time-resolved snapshots in the LSB region for the wake cases
2. Apply entropy-production-weighted SPOD (from Goal 001 H3 — reuse same implementation)
3. Identify: (a) wake-passing frequency `f_wake`, (b) KH frequency `f_KH`, (c) streaky structure signature (broadband, low frequency)
4. Track how the SPOD eigenvalue spectrum changes with incidence
5. At i = +12°: does KH mode persist despite wake impingement? Is its frequency modified?
6. At i = 0°: does the KH mode disappear (bubble suppressed) and only streak modes remain?

**Expected result:** At i = 0° with wakes, the KH mode (dominant in clean-inflow case) is weakened and the wake-passing harmonic becomes the dominant SPOD mode. At i = +12°, both KH and wake-passing modes are present — a competition. The entropy-weighted inner product ensures only the loss-relevant structures appear in the top modes.

**Success criteria:** SPOD mode shapes are physically identifiable (KH = spanwise-coherent roll-up; streak = streamwise-elongated, alternating sign in span). Clear frequency separation between wake-passing and KH modes by at least a factor of 2.

**Config:** `configs/goals/002/h002-spod-wake-LSB.yaml`

---

## Hypothesis 3: Intermittency and Turbulent Spot Production Rate

**Status:** PLANNED

**Reasoning:** Wake-induced transition produces turbulent spots during the wake passage. The spot production rate Ṅ is a key parameter for transition models. This has been studied on flat plates (Genoa group) but never at off-design LPT incidence. If the spot production rate can be correlated with wake parameters and incidence, it provides a direct input to improved transition models.

**Analysis tasks:**
1. Apply wavelet-based intermittency detection (following Genoa group methodology) to time-resolved surface Cf from DNS
2. Identify turbulent spots: onset location, convective velocity, spreading angle
3. Compute spot production rate Ṅ(x) as a function of chordwise position for each wake case and incidence
4. Compare Ṅ between clean-inflow and wake cases
5. Correlate Ṅ with: wake momentum thickness θ_w, wake convective time τ_w = c_ax/U_in, incidence angle i

**Expected result:** Wake passage triggers spots earlier and at higher production rate than clean-inflow bypass transition. At i = +12°, the bubble delays spot convection — spots must travel through the separated region, which may suppress or modify them. The spot production rate correlation provides data for improved Langtry-Menter transition model constants at off-design.

**Success criteria:** Intermittency factor γ(x,t) phase-averaged over wake cycles shows clear turbulent spot signatures. Spot production rate Ṅ can be estimated for at least 2 incidence angles with and without wakes.

**Config:** `configs/goals/002/h003-intermittency-spots.yaml`

---

## Paper Structure (Proposed)

### Conference paper (Turbo Expo 2026): H0 + H1
Focus on the differential entropy budget and wake effectiveness across incidence — the engineering-relevant result.

### Journal extension (JTM): Full paper H0–H3
Add SPOD coherent structure analysis and intermittency/spot production rate for the complete physical picture.

1. Introduction — gap: no DNS study of wake effects on entropy budget in off-design LPT
2. Computational setup — wake injection methodology (DHIT precursor, Strouhal number selection)
3. Baseline comparison: clean inflow vs. wakes at i = 0° (differential budget) — H0
4. Incidence effect: wake effectiveness across i = 0°, ±12° — H1
5. Coherent structures: SPOD of wake-LSB interaction — H2
6. Turbulent spot production: intermittency analysis — H3
7. Discussion: implications for transition model parameters (Langtry-Menter Flength, Fonset corrections)
8. Conclusions

---

## Dependency on Goal 001

This goal builds directly on Goal 001:
- The entropy budget framework (analysis code, region definitions) is reused
- The entropy-weighted SPOD implementation from Goal 001 H3 is reused
- The clean-inflow DNS results from Goal 001 are the reference baseline for every differential comparison
- Goal 001 must be substantially complete before the differential budget analysis in H0 is meaningful
