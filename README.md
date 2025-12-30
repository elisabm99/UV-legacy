# UV-legacy: HST/FOS stacked spectra in detector-count units
Data products created starting from HST FOS spectra that are part of the project SV2965. The underlying data products are publicly available on [MAST](https://mast.stsci.edu/search/ui/#/).

This repository provides stacked HST/FOS spectra in **detector-count units** (counts per spectral pixel) together with the wavelength grid, a data-quality mask, model dark background, and a precomputed **forward response kernel** that maps an input spectral flux model into expected detector counts on the same grid.

These stacks were produced for the analysis in:  
[2506.19962](https://arxiv.org/abs/2506.19962)

---

## Contents

- `stacks/*.fits`  
  Stacked spectra in the format described below.

---

## File naming

Each stack corresponds to a single line of sight (LOS) and is named

- `LLLLLL_stack_cnts.fits`

where `LLLLLL` is the LOS identifier (the first 6 characters of the original exposure rootnames used in this analysis).

---

## FITS format

Each `*.fits` contains:

### HDU 0: PRIMARY
Header only (no image data). It stores instrument metadata copied from the source exposures plus stack bookkeeping keywords (see below).

### HDU 1: `SCI` (binary table)
The main stacked spectrum.

Columns:

| Column | Type | Units | Meaning |
|---|---:|---|---|
| `WAVELENGTH` | float64 | Angstrom | Wavelength grid (FOS pixel grid; length `NPIX`) |
| `COUNTS` | float64 | count | Total stacked counts per pixel (object + background) |
| `BKG_COUNTS` | float64 | count | Stacked dark background counts per pixel (post flat-field convention used in the pipeline) |
| `DQ` | int16 | — | Data-quality flag per pixel |

**Stacking convention**
- `COUNTS` and `BKG_COUNTS` are summed across exposures only where that exposure has `DQ == 0` at that pixel.
- The stacked `DQ` is the **bitwise OR** across input exposures (any flagged pixel remains flagged).
- Wavelength grids are required to match across inputs (within numerical tolerance).

### HDU 2 (optional): `KERNEL` (binary table)
This stores the precomputed response kernel `R(λ)` for the stack described in 2506.19962.

Columns:

| Column | Type | Units | Meaning |
|---|---:|---|---|
| `WAVELENGTH` | float64 | Angstrom | Kernel wavelength grid (matches the science grid over the kernel length) |
| `R` | float64 | count / (flux unit) | Response kernel such that `lambda = flux_model * R` gives expected counts |


## Header keywords (stack bookkeeping)

The PRIMARY header includes (at minimum) the following stack-level bookkeeping keywords:

- `NSTACK` : number of input files combined into the stack.
- `EXPPIX` : total exposure time per pixel (seconds) for a representative “middle” pixel summed across exposures.

⚠️ Notes on exposure:
The stacks also carry additional keywords inherited from the first exposure that entered the stack (e.g. `DETECTOR`, `FGWA_ID`, etc.). Only keywords that are meaningful at the stack level should be interpreted literally; see “Provenance” below for recommended handling of per-exposure metadata.

---

## Data quality (`DQ`)

`DQ` values are inherited from the original FOS calibration products and combined conservatively via bitwise OR. 

---

