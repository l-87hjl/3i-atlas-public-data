# 3i-atlas-public-data

Public datasets referenced in my Substack and Medium analyses of the
interstellar object **3I/ATLAS**. Files include observational outputs,
solution-level data, and related ephemeris tables. Provided for
transparency, reproducibility, and independent review.

---

## Overview

This repository provides public access to ephemeris data, solution-specific
outputs, and supporting documentation used in analyses of the interstellar
object designated **3I/ATLAS**.

The repository is structured to support transparency, reproducibility, and
independent review. Data are organized by JPL solution, with global
methodological documentation maintained separately to ensure consistency
across analyses.

---

## Repository Structure

### `/documentation/`

Global methodological reference files applicable to all solutions, including:

- Ephemeris field definitions
- Interpretation of sigma-based positional uncertainty
- Statistical usage notes for JPL Horizons outputs

These documents define *what* data are extracted and *how* uncertainty
quantities must be interpreted.

---

### `/JPL_Solutions/`

Solution-specific data directories corresponding to individual JPL orbital
solutions (e.g., Sol44, Sol45).

Each solution directory may contain:
- Raw JPL Horizons output files
- Cleaned, machine-readable CSV derivatives
- Query parameter records
- Solution-specific notes or metadata

Where applicable, solution directories may also include a brief README
describing that solution’s scope or distinguishing characteristics.

---

## Data Philosophy

- Raw source files are preserved unchanged.
- Derived files are generated deterministically from raw outputs.
- Filenames encode solution number, observatory code, and time span.
- Previously published files are never overwritten; updates are additive.

This repository provides source data only and does not present conclusions.

---

## Usage

All files are publicly accessible and may be downloaded directly.
No authentication or specialized tools are required.

Raw download links follow the standard GitHub format:

https://raw.githubusercontent.com/l-87hjl/3i-atlas-public-data/main/<path>

---

## Scope and Limitations

- Data reflect the assumptions and force models of the corresponding JPL
  solutions at time of generation.
- Formal uncertainty values assume Gaussian error propagation and may not
  capture unmodeled systematics.
- Inclusion of data does not imply physical interpretation or anomaly.

---

## License and Attribution

Unless otherwise stated, contents are shared for public, non-restricted use.
Attribution is appreciated when referencing or redistributing these materials.

Source data include publicly available outputs from NASA/JPL Horizons and
Minor Planet Center publications. No affiliation or endorsement is implied.
