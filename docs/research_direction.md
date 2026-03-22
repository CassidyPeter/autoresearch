# Research Direction: Off-Design Loss in Low-Pressure Turbines

## The Core Question

**Can off-design loss be characterised in a physical way?**

Specifically: how does entropy production shift between boundary layer regions as incidence changes, and why does RANS fail to capture it?

---

## Why This Is Novel

The Przytarski & Wheeler (2021) entropy budget framework — decomposing loss via the dissipation coefficient C_D into turbulent production (Pr) and viscous dissipation — has only been applied to compressor cascades. Wheeler (2018) and Spencer (2023) are the only two references for non-equilibrium BL analysis in turbomachinery, and both are compressors. No one has:

1. Applied an entropy budget decomposition to an **off-design LPT** at varying incidence
2. Characterised the **non-equilibrium turbulent BL** in LPT context (only compressor literature exists)
3. Used **SPOD to resolve frequency-specific coherent structures** (KH, TS) inside an LPT laminar separation bubble across an incidence sweep
4. Studied **wake-induced transition effects on entropy budgets** in off-design LPT via DNS

The Senior & Miller (2024) data-centric approach and Coull & Hodson (2012) physical model are both on-design only. Off-design has no physical loss breakdown in the literature.

---

## Two Papers

### Paper 1 — Journal (JTM target, Y3/Y4)
**"Entropy Production and Non-Equilibrium Boundary Layer Physics in Off-Design Low-Pressure Turbine Flows"**

Data source: DNS/iLES at i = 0°, +12°, −12° (clean inflow, already running)
Analysis framework: Przytarski & Wheeler entropy budget applied to LPT first time
→ See `docs/goals/001-entropy-budget-off-design-LPT.md`

### Paper 2 — Conference (Turbo Expo 2026) → Journal
**"Wake-Induced Transition and Its Effect on Entropy Production in Off-Design LPT Flows"**

Data source: DNS/iLES with incoming periodic wakes at i = 0°, ±12° (Y3 simulations)
Analysis framework: differential entropy budget (wakes vs. clean), SPOD of wake-LSB interaction
→ See `docs/goals/002-wake-induced-transition.md`

---

## Reduced-Order Model Strategy

The ultimate model deliverable (Objective 2 of the FRIA) is a physics-based off-design loss predictor. Given the challenges with POD (too many modes for turbulent breakdown) and the novelty gaps found in literature:

**Recommended approach — two-tier ROM:**

**Tier 1: Loss model (interpretable, publishable)**
- Input: incidence angle, Re, FSTI, integral length scale
- Output: loss breakdown fractions by region (laminar BL, LSB, non-eq. turbulent BL, wake)
- Method: Genetic programming / symbolic regression on combined DNS + experimental data
- Rationale: fully interpretable, closed-form equations, validated against VKI data repository
- Gap filled: no symbolic regression approach exists for off-design LPT loss prediction

**Tier 2: Field ROM (optional, higher risk)**
- Encode the entropy production rate field (not velocity field) via entropy-weighted SPOD
- The entropy field is lower-dimensional than the velocity field and more directly linked to loss
- Entropy-weighted inner product for SPOD: `<f,g>_s = ∫ ṡ_a * f * g dΩ`
- Gap filled: no loss-weighted SPOD or autoencoder exists in the literature

The Tier 2 approach is novel enough for a methods paper if it works, or can be a section of Paper 2.

---

## Why NOT to Use Standard POD for the Velocity Field

Peter identified this correctly. The turbulent breakdown region requires O(100s) of POD modes because:
- Energy is spread broadly across wavenumbers post-breakdown
- POD modes mix frequencies (no spectral selectivity)
- The high-dimensionality comes from the broadband inertial cascade

SPOD resolves this partially (frequency-selective) but still requires many modes in the turbulent regime. Mode-decomposing autoencoders (Murata et al. 2020, Fukami et al.) could compress this to O(10) modes but have never been applied to LPT flows. This is a viable future direction but not needed for Paper 1 or 2.

---

## Analysis Pipeline (for DNS data)

Quantities to extract from DNS fields (in priority order):

1. **Entropy generation rate field** `ṡ = Φ/T` — integrate by region for loss audit
2. **Dissipation coefficient** `C_D = Pr + D_visc` — along suction and pressure surfaces
3. **Turbulence production** `Pr = -(u'_i u'_j) * ∂U_i/∂x_j / U_e³` — wall-normal integral
4. **Viscous dissipation** `D_visc = ν/U_e³ * ∫(∂U_i/∂x_j)² dy` — wall-normal integral
5. **Non-equilibrium indicator**: divergence of `Pr` and `D_visc` along blade surface
6. **Reynolds stress anisotropy tensor** `b_ij = u'_iu'_j / (2k) - δ_ij/3` — for RANS failure mode mapping
7. **SPOD** of the entropy generation rate field — modes at Strouhal numbers corresponding to KH and TS frequencies

---

## Connections to Experimental Data

| DNS quantity | Experimental proxy |
|---|---|
| C_D along surface | Hot-film quasi-wall-shear + TR-PIV Reynolds stresses |
| Turbulence production field | TR-PIV `<u'v'>` and `∂U/∂y` |
| Transition location | Hot-film intermittency + wavelet detection |
| LSB length and height | TR-PIV mean flow + surface pressure Cp |
| Wake loss | Downstream 5HP total pressure |
| Reynolds stress anisotropy | Triple-wire hot-wire (anisotropy) |

The experimental campaign (54 conditions, Re = 80k/160k, i = 0°→±15°) provides validation across a much broader parameter space than the DNS can cover — this is the main contribution of the combined approach.
