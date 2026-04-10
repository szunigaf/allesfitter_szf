# Binary Host Star Fitting in allesfitter (fork)

This document describes the extension to allesfitter that enables fitting transits
of planets orbiting **different stars in a blended binary system**, along with flares
assignable to a specific host star.

---

## Background

In a blended binary system (stars A and B unresolved in the aperture), the observed
transit depth is diluted by the light from the non-transited star.  The standard
binary flux ratio is:

$$\beta = \frac{F_B}{F_A} \qquad (\text{fainter over brighter, } 0 < \beta \lesssim 1)$$

The dilution factors for each star are:

$$D_A = \frac{\beta}{1+\beta} \qquad D_B = \frac{1}{1+\beta}$$

where $D_X$ is the fraction of total in-aperture flux that does **not** come from
the transited star.  Note that $D_A + D_B = 1$ always, and neither equals $\beta$
directly.  $\beta$ is wavelength-dependent — use one `flux_ratio_FILTER` per filter
band.

---

## Changes Made to the allesfitter Source Code

### `allesfitter/basement.py`

| Change | Purpose |
|--------|---------|
| `companion_host` dict built from `companion_X_host` settings | Maps each planet to `host_A` or `host_B` |
| `inst_filter` dict built from `inst_filter_INST` settings | Groups instruments by filter band |
| `flare_host` dict built from `flare_N_host` settings | Assigns each flare to a host star |
| `validate('dil_COMPANION_INST', ...)` added to param loop | Registers per-companion dilution as optional parameter |
| `coupled_with` resolver extended with `~` prefix | `~flux_ratio_r` means $1 - \beta/(1+\beta)$ |

All new settings are **optional with backwards-compatible defaults** — existing
single-star fits require no changes.

### `allesfitter/computer.py`

| Change | Purpose |
|--------|---------|
| `flux_ratio_FILTER` derived-dilution block in `update_params()` | Auto-computes `dil_COMPANION_INST` from $\beta$ for all affected companions/instruments |
| `coupled_with` complement resolver mirrored here | Consistent with `basement.py` |
| `flux_subfct_ellc()` dilution lookup | Uses `dil_COMPANION_INST` if present, else `dil_INST` |
| `flux_fct_piecewise()` dilution lookup (TTV path) | Same fallback for `light_3` argument |
| `flux_subfct_flares()` dilution lookup | Uses `dil_flare_host_INST` based on `flare_N_host` |

---

## New Optional Settings

### `settings.csv`

```
# Assign each companion to a host star.
# 'host_A' = bright star (default, backwards-compatible alias for 'host')
# 'host_B' = faint star
companion_b_host,    host_A
companion_d_host,    host_B

# Map each instrument to its filter band label.
# Instruments sharing the same label use the same flux_ratio_FILTER.
# Default: each instrument is its own band (no sharing).
inst_filter_TTT_r,   r
inst_filter_HCAM_r,  r
inst_filter_TTT_g,   g
inst_filter_HCAM_g,  g

# Assign each flare to a host star (default: host_A).
flare_1_host,        host_A
```

### `params.csv`

```
# One flux_ratio per filter band  (beta = F_B / F_A, fainter / brighter)
# This is the ONLY new free parameter needed.
# All dil_COMPANION_INST values are derived automatically.
#
#name,          value,  fit,  bounds,       label,        unit
flux_ratio_r,   0.43,   1,    uniform 0 5,  $\beta_r$,
flux_ratio_g,   0.30,   1,    uniform 0 5,  $\beta_g$,
```

No `dil_` entries are required when `flux_ratio_FILTER` is present.
If no `flux_ratio_*` key is found, the code uses existing `dil_INST` entries
unchanged — **single-star fits are unaffected**.

---

## Complete Example: Two Planets, One Flare

### Scenario

- **Star A** (bright): planet b transits it; flare 1 originates on it
- **Star B** (faint): planet d transits it
- Two instruments: `TTT_r` (r-band, used for planet b) and `HCAM_r` (r-band, used for planet d)
- Binary flux ratio in r-band: $\beta_r \approx 0.43$ (to be fitted)

### `settings.csv`

```
companions_phot,       b d
inst_phot,             TTT_r HCAM_r

# Host assignment
companion_b_host,      host_A
companion_d_host,      host_B

# Filter mapping (both instruments use r-band → share flux_ratio_r)
inst_filter_TTT_r,     r
inst_filter_HCAM_r,    r

# Flare on star A
N_flares,              1
flare_1_host,          host_A

# Standard allesfitter settings below
...
baseline_flux_TTT_r,   sample_offset
baseline_flux_HCAM_r,  sample_offset
error_flux_TTT_r,      sample
error_flux_HCAM_r,     sample
```

### `params.csv`

```
#name,                  value,      fit,  bounds,               label,                unit
# ---- binary flux ratio (one free parameter for r-band) ----
flux_ratio_r,           0.43,       1,    uniform 0 5,          $\beta_r$,

# ---- planet b (orbits star A) ----
b_rr,                   0.12,       1,    uniform 0.01 0.5,     $R_b/R_A$,
b_rsuma,                0.08,       1,    uniform 0.01 0.5,     $(R_A+R_b)/a_b$,
b_cosi,                 0.0,        1,    uniform 0 1,          $\cos i_b$,
b_epoch,                2458900.0,  1,    uniform 2458899 2458901, $T_{0,b}$,        BJD
b_period,               3.5,        1,    uniform 3.0 4.0,      $P_b$,              days
b_sbratio_TTT_r,        0.0,        0,    uniform 0 1,          $J_{b,\rm TTT_r}$,
host_ldc_q1_TTT_r,      0.4,        1,    uniform 0 1,          $q_{1,\rm TTT_r}$,
host_ldc_q2_TTT_r,      0.3,        1,    uniform 0 1,          $q_{2,\rm TTT_r}$,
ln_err_flux_TTT_r,      -6.0,       1,    uniform -10 0,        $\ln\sigma_{\rm TTT_r}$,

# ---- planet d (orbits star B) ----
d_rr,                   0.15,       1,    uniform 0.01 0.5,     $R_d/R_B$,
d_rsuma,                0.10,       1,    uniform 0.01 0.5,     $(R_B+R_d)/a_d$,
d_cosi,                 0.0,        1,    uniform 0 1,          $\cos i_d$,
d_epoch,                2458902.0,  1,    uniform 2458901 2458903, $T_{0,d}$,        BJD
d_period,               5.2,        1,    uniform 4.5 6.0,      $P_d$,              days
d_sbratio_HCAM_r,       0.0,        0,    uniform 0 1,          $J_{d,\rm HCAM_r}$,
host_ldc_q1_HCAM_r,     0.4,        1,    uniform 0 1,          $q_{1,\rm HCAM_r}$,
host_ldc_q2_HCAM_r,     0.3,        1,    uniform 0 1,          $q_{2,\rm HCAM_r}$,
ln_err_flux_HCAM_r,     -6.0,       1,    uniform -10 0,        $\ln\sigma_{\rm HCAM_r}$,

# ---- flare 1 (on star A, observed in TTT_r) ----
# Dilution for this flare is taken from dil_host_A_TTT_r,
# which is derived as beta/(1+beta) from flux_ratio_r automatically.
flare_tpeak_1,          2458900.5,  1,    uniform 2458900 2458901, $t_{\rm peak,1}$, BJD
flare_fwhm_1,           0.005,      1,    uniform 0.0001 0.1,   $\rm FWHM_1$,       days
flare_ampl_1,           0.02,       1,    uniform 0 1,          $A_1$,

# ---- baseline / jitter ----
baseline_flux_TTT_r,    0.0,        1,    uniform -0.1 0.1,     $\Delta F_{\rm TTT_r}$,
baseline_flux_HCAM_r,   0.0,        1,    uniform -0.1 0.1,     $\Delta F_{\rm HCAM_r}$,
```

> **Note on `d_rr`**: This is $R_d / R_B$ (planet radius over the *transited* star
> radius), not over the total system.  The dilution factor $D_B = 1/(1+\beta)$
> automatically corrects the observed depth so that the fitted `d_rr` is the true
> planet-to-star radius ratio for star B.

### `run.py`

```python
import allesfitter

# Nested sampling fit
allesfitter.ns_fit('path/to/fit_folder')
allesfitter.ns_output('path/to/fit_folder')
```

---

## How the Dilution Is Applied Internally

When `flux_ratio_r = 0.43` is sampled at each MCMC/NS step, `update_params()` computes:

$$D_b = \frac{0.43}{1.43} \approx 0.301 \quad \Rightarrow \quad \texttt{dil\_b\_TTT\_r} = 0.301$$

$$D_d = \frac{1}{1.43} \approx 0.699 \quad \Rightarrow \quad \texttt{dil\_d\_HCAM\_r} = 0.699$$

These override any manually set `dil_` entries for those companion/instrument pairs.
The flare model for flare 1 uses `dil_host_A_TTT_r = dil_b_TTT_r = 0.301` because
`flare_1_host = host_A`.

---

## Backwards Compatibility

| User scenario | Required changes to config | Code behaviour |
|---------------|--------------------------|---------------|
| Single star, no `flux_ratio_*` | None | Identical to original allesfitter |
| Single star, explicit `dil_INST` | None | Identical to original allesfitter |
| Binary, fixed dilution per companion | Add `dil_b_INST` and `dil_d_INST` to `params.csv` | Per-companion dilution used |
| Binary, fitting $\beta$ per filter | Add `flux_ratio_FILTER` to `params.csv` + `companion_X_host` and `inst_filter_INST` to `settings.csv` | Full binary mode |

---

## Parameter Reference

| Parameter | Location | Type | Description |
|-----------|----------|------|-------------|
| `companion_X_host` | `settings.csv` | Optional | Host star for companion X: `host_A` (default) or `host_B` |
| `inst_filter_INST` | `settings.csv` | Optional | Filter band label for instrument INST (default: `INST`) |
| `flare_N_host` | `settings.csv` | Optional | Host star for flare N: `host_A` (default) or `host_B` |
| `flux_ratio_FILTER` | `params.csv` | Optional free param | $\beta = F_B/F_A$ in the given filter band |
| `dil_COMPANION_INST` | `params.csv` | Optional fixed/free | Per-companion dilution; overrides `dil_INST` for that companion only |

---

*This extension was designed and implemented for the `allesfitter_szf` fork (April 2026).*
*Cite the original allesfitter package (Günther & Daylan 2021) for the base code.*
