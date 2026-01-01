3I/ATLAS O–C Pipeline: Pre-Math Failure Modes and Pre-Flight Checks
Scope: This document enumerates the most common places an O–C (Observed minus Computed) residual pipeline can go wrong when comparing MPC/MPEC astrometry to JPL Horizons predicted astrometry + uncertainty fields.
Goal: Identify failure modes, what they look like, and how to detect/avoid them before validating calculations.
0) Canonical Inputs and Definitions
Required Horizons fields
The pipeline must extract at least the following fields from Horizons ephemerides:
UTC time
RA (ICRF) and DEC (ICRF)
dRA·cos(DEC), d(DEC)/dt (optional but useful)
Uncertainty fields: RA_3SIGMA, DEC_3SIGMA, SMAA_3SIG, SMIA_3SIG, THETA
Source definition: EPHEMERIS FIELD SPECIFICATION — 3I/ATLAS �
EPHEMERIS_FIELD_SPEC_3I_ATLAS.txt None
Correct uncertainty interpretation
RA_3SIGMA / DEC_3SIGMA are formal ±3σ covariance-propagated uncertainties, not measurement error.
Correct positional consistency testing uses the rotated plane-of-sky error ellipse (SMAA_3SIG, SMIA_3SIG, THETA).
RA_3SIGMA and DEC_3SIGMA must not be treated as independent axis-aligned bounds.
Source definition: SIGMA UNCERTAINTY INTERPRETATION — JPL HORIZONS �
SIGMA_UNCERTAINTY_INTERPRETATION_JPL_HORIZONS.txt None
1) Identity and Scope Mismatches
1.1 Wrong target / designation / SPK-ID
What happens: You query the wrong object or an ambiguous target resolution changes between runs.
Symptoms: Residuals are “catastrophically wrong” for everything, but the pipeline looks mechanically correct.
What to check
Log and store the Horizons header block for each pull and confirm the object identity lines are stable.
Confirm the object name/designation matches what your MPEC observation table refers to.
Prevention
Store a canonical TARGET_ID string for your project (whatever is least ambiguous for Horizons in your workflow).
Do not allow free-text object names in scripts without a pinned ID.
1.2 Wrong solution context (Sol40 vs Sol42 vs Sol44 etc.)
What happens: Your Horizons output is implicitly tied to the “current” published solution, which may have advanced since your analysis intent.
Symptoms: You think you’re testing Sol44, but the ephemeris was generated under Sol45+.
What to check
Horizons output headers include solution metadata (e.g., “soln ref”, “Soln.date”). Capture and store them with the data.
If you have multiple solutions in the repo, ensure each dataset is stored under the correct SolXX/ node.
Prevention
Always save the Horizons header with your ephemeris extraction.
Treat each SolXX folder as immutable historical snapshots.
2) “Observer / Center” Misuse (Topocentric vs Geocentric vs Planet-centric)
2.1 Using numeric centers when you intended an observatory code
What happens: You accidentally use CENTER=500@399 or other numeric centers when you meant “as seen from observatory G96/Q14/etc.”
Symptoms: Systematic offsets that vary by site; parallax-like errors; some sites appear wildly wrong.
What to check
For topocentric observatory predictions, the Horizons query should use only the 3-character MPC observatory code (e.g., G96, Q14) as the observer center.
Confirm the header states the observer is that observatory site (not “Earth geocenter” or “Mars barycenter”).
Prevention
Enforce a strict validation rule in tooling:
“Observer-mode pulls must have CENTER matching /^[A-Z0-9]{3}$/.”
If you want geocenter intentionally, store those files separately and label them explicitly.
3) Time Handling Failure Modes (High-Impact)
3.1 UTC vs TT/TDB confusion
What happens: One dataset is interpreted in UTC, another in TT/TDB.
Symptoms: Smooth, persistent along-track errors; “almost right but not quite” everywhere.
What to check
Ensure both sides of the comparison are expressed in the same time scale.
Confirm Horizons output UTC field is used when matching MPEC observation timestamps.
Prevention
Store a field time_scale for each dataset (“UTC”, “TDB”, etc.). Do not infer.
3.2 Timestamp parsing mistakes (locale, formats, day/month inversion)
What happens: Parsing libraries interpret dates incorrectly.
Symptoms: Large jumps, off-by-days, or inconsistent mismatches across date boundaries.
What to check
Round-trip parse test: parse → format back → compare to original.
Confirm the “first few” and “last few” timestamps look exactly like expected.
Prevention
Read timestamps as strings first; parse with a single explicit format; never rely on locale inference.
3.3 Step-size mismatch and interpolation hazards
What happens: Horizons ephemerides are generated at coarse steps (e.g., 1 min, 1 hr, 12 hr), but you need predictions at exact observation times.
Symptoms: Results change noticeably when you regenerate with a different step size.
What to check
For each observation, log the matched ephemeris timestamp and the absolute Δt (seconds).
Compare runs at multiple step sizes and confirm residuals are stable under method.
Prevention
Best: query Horizons exactly at observation times (TLIST style).
If interpolating, document interpolation method and validate against a “dense sampling” truth set.
3.4 Precision padding “fake precision” (critical)
What happens: Your observation timestamp column appears to have microseconds (e.g., .000000), but those digits are padding, not measurement resolution.
Symptoms: Nearest-neighbor matching “chooses wrong row” due to false confidence; systematic drift or bias introduced.
What to check
Histogram fractional seconds: if most rows end in the same trailing digits (e.g., .000000), treat it as formatted, not measured.
Track the original observational precision source (MPEC formatting or fractional day resolution).
Prevention
Maintain two fields:
obs_time_value (datetime used for matching)
obs_time_precision (e.g., “1 sec”, “0.1 sec”, “0.001 day”, “unknown”)
When presenting timestamps, render only to known precision and avoid padded digits in “canonical” columns.
4) Coordinate and Units Mismatches
4.1 RA conversion errors (hours↔degrees, missing cos(dec))
What happens: RA residuals are computed incorrectly due to unit confusion or missing cos(dec) scaling.
Symptoms: RA residuals are off by factors of 15 or vary strangely with declination.
What to check
Ensure the pipeline uses ΔRA·cos(DEC) when converting RA separations into on-sky arcseconds.
Confirm RA is consistently interpreted as sexagesimal hours (if provided that way).
Prevention
Store intermediate values explicitly (RA_deg, DEC_deg, dRAcosD_arcsec, etc.) to make auditing trivial.
4.2 Frame mismatch (ICRF/J2000 vs other frames)
What happens: Predicted and observed coordinates are expressed in different frames.
Symptoms: Residuals have a coherent systematic offset/rotation.
What to check
Confirm Horizons output RA/DEC is ICRF/J2000 when matching MPC astrometry.
Ensure you are not mixing ecliptic elements readouts with equatorial astrometry.
Prevention
Hardcode the expected frame in the Horizons query and store it in metadata.
5) Uncertainty Field Misuse (Sigma / Ellipse)
5.1 Reading the wrong columns (mis-mapped extraction)
What happens: Columns shift or extraction logic grabs wrong uncertainty fields.
Symptoms: sigma counts appear nonsensical or constant across rows.
What to check
Verify the extracted columns match the field spec order and labels. �
EPHEMERIS_FIELD_SPEC_3I_ATLAS.txt None
Confirm SMAA_3SIG, SMIA_3SIG, THETA are present whenever “ellipse testing” is claimed.
Prevention
Use label-based extraction, not positional indexing, where possible.
Validate column names against an allowed list.
5.2 Treating RA_3SIGMA and DEC_3SIGMA as independent bounds
What happens: You do a naive axis-aligned “within 3σ” test.
Symptoms: pass/fail results disagree with ellipse-based tests; diagonal residuals behave inconsistently.
What to check
Confirm ellipse fields are used and residual vector is tested in the rotated ellipse coordinates. �
SIGMA_UNCERTAINTY_INTERPRETATION_JPL_HORIZONS.txt None
Prevention
Require ellipse fields for any “sigma count” reporting. If absent, report “sigma unavailable”.
5.3 THETA convention mistakes
What happens: Rotation is applied with wrong sign or reference axis.
Symptoms: ellipse test behaves “rotated wrong”; false failures/passes.
What to check
THETA definition: clockwise from +RA toward semi-major axis, in direction of +DEC. �
SIGMA_UNCERTAINTY_INTERPRETATION_JPL_HORIZONS.txt None
Prevention
Unit tests: feed a synthetic residual aligned with the semi-major axis and confirm normalized distance behaves as expected.
6) Joins, Indexing, and Bookkeeping
6.1 Wrong join key or non-unique join
What happens: One observation is joined to the wrong prediction row.
Symptoms: RA/DEC look “swapped” across neighbors; anomalies cluster at boundaries.
What to check
Ensure join key uniqueness (timestamp + observatory code + row id if needed).
Log every join with: obs timestamp, matched ephem timestamp, Δt.
Prevention
Enforce uniqueness constraints in code; reject joins that match multiple ephemeris records.
6.2 Sorting mistakes (string sort vs datetime sort)
What happens: Data is ordered incorrectly, and “nearest time” logic fails.
Symptoms: periodic spikes or discontinuities.
What to check
Confirm sorting by parsed datetime, not string.
Confirm monotonic time increasing after sorting.
Prevention
Always parse time, then sort; never sort raw timestamp strings.
6.3 Silent truncation / type coercion
What happens: CSV readers truncate precision or misread scientific notation.
Symptoms: trailing digits disappear; uncertainty fields become zeros; columns become NaN.
What to check
Read as string first, then convert with explicit rules.
Confirm summary stats (min/max) are non-degenerate.
Prevention
Schema enforcement at ingest.
7) Minimal Pre-Flight Checklist (Recommended to Run Every Time)
Before doing any residual math:
Identity pinned: target and solution metadata captured from Horizons header
Observer correct: observatory-code center used for topocentric comparisons
Time scale consistent: both sides in UTC (or explicitly reconciled)
Time precision acknowledged: no padded zeros masquerading as measurement precision
Match audit logged: for each row, store obs_time, matched_ephem_time, Δt
Field completeness: required uncertainty ellipse fields present when sigma testing is claimed �
EPHEMERIS_FIELD_SPEC_3I_ATLAS.txt None
Join uniqueness: join key produces exactly one match per observation
