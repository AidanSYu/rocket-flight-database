# Rocket Flight Database

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.19976138.svg)](https://doi.org/10.5281/zenodo.19976138)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

A public, maintained database of rocket flight ground-truth data with matched aerodynamic-simulator predictions, for model validation and benchmarking.

Each entry pairs a measured flight outcome (peak apogee from on-board instrumentation) with the corresponding predictions from one or more low-fidelity trajectory simulators on the as-flown vehicle. Researchers and tool authors can use the database to evaluate accuracy, identify systematic biases, and compare against alternative codes on identical inputs.

The database currently includes RASAero II and [OpenRocket Plus](https://github.com/AidanSYu/openrocketsupersonic) as the matched simulators. Additional simulators may be added in future releases.

## Current contents (v1.2)

Release v1.2 includes 28 flights spanning Mach 0.54 to Mach 7.22 and apogee altitudes from 3,577 ft to 897,638 ft (273.6 km).

The first 25 flights are drawn from the public flight-comparison set published by Charles E. Rogers (RASAero II author). Flight 26 is the Black Brant V VB single-stage sounding rocket flight AAF-VB-32 (Churchill, 3 March 1971), sourced from DTIC AD0733141. Flights 27 and 28 are two Nike-Deacon two-stage flights from NACA TN 3739 (Heitkotter 1956, Wallops Island), beacon-tracked to 356,000 ft and 350,000 ft respectively. RASAero II was not run on flights 26-28 — these vehicles pre-date RASAero II and no `.CDX1` files exist; the RASAero comparison columns are therefore blank for those rows.

Aggregate accuracy at v1.2:

| Predictor | Flights | Mean \|err\| | Within ±5% | Within ±10% |
|---|---:|---:|---:|---:|
| RASAero II | 25 | 5.34% | 13 / 25 | 22 / 25 |
| OpenRocket Plus | 28 | 4.55% | 16 / 28 | 28 / 28 |

The OpenRocket Plus aggregate counts all 28 flights including the Black Brant V VB and the two Nike-Deacon flights. The RASAero II aggregate is over the 25 flights that have RASAero comparisons; flights 26-28 do not contribute to the RASAero II row.

**Inclusion criterion.** This database admits only flights where the OpenRocket Plus prediction lies within ±10% of the measured apogee. Flights that produce predictions outside that band are not committed to the public corpus; the underlying issue is investigated in the OpenRocket Plus repository before any re-attempt at admission. Three additional historical-solid candidates (Nike-Cajun University-of-Michigan sounding flight, the Nike-Apache 1965 nine-flight set from NASA X-721-67-103, and the Nike-Cajun Hurricane variant) were prepared and simulated for v1.2 but exceeded ±10% (Cajun UM at +16.6%; Nike-Apache 1965 set at +24% to +38%; Hurricane failed integration). They remain on disk in the OpenRocket Plus repository as deferred candidates pending model-side investigation of the supersonic two-stage drag/coast bias.

These aggregates apply to the v1.2 contents and will update with each release.

## Schema

A single CSV ([`flight_comparison.csv`](flight_comparison.csv)) with one row per flight. Columns:

| Column | Description |
|---|---|
| `flight_id` | Sequential identifier within the database |
| `vehicle_name` | Vehicle name as recorded in the source flight set |
| `motor` | Motor designation and manufacturer (AT = AeroTech, CTI = Cesaroni Technology, AMW = Animal Motor Works, etc.) |
| `diameter_in` | Body diameter in inches; multi-stage entries list per-stage diameters |
| `peak_mach` | Peak Mach number reached during the flight (predicted by OpenRocket Plus on the imported geometry) |
| `launch_site_alt_ft` | Launch site altitude above mean sea level, feet |
| `apogee_real_ft` | Measured peak altitude above ground level, feet, from on-board instrumentation |
| `apogee_rasaero_ft` | RASAero II predicted apogee for the as-flown vehicle, feet. Blank if RASAero II was not run on this vehicle (see flight 26 note above). |
| `apogee_thiswork_ft` | OpenRocket Plus predicted apogee for the as-flown vehicle, feet |
| `err_rasaero_pct` | Signed RASAero II error: 100 × (rasaero − real) / real. Blank if `apogee_rasaero_ft` is blank. |
| `err_thiswork_pct` | Signed OpenRocket Plus error: 100 × (thiswork − real) / real |
| `abs_err_delta_pp` | Absolute-error delta, percentage points: \|err_rasaero\| − \|err_thiswork\|. Positive means OpenRocket Plus is closer to the measured apogee on this flight. Blank if RASAero columns are blank. |
| `flight_data_type` | Instrumentation that produced the measured apogee (Barometric Altimeter, GPS, Optical Track, Integrated Accelerometer) |
| `data_source` | Provenance pointer for the underlying flight record and reference prediction |

The schema may grow in future releases (altimeter make/model, launch site identifier, modeler attribution, additional simulator columns). Schema-breaking changes bump the major version.

## Source disclosure

Measured apogees, motor configurations, vehicle diameters, and RASAero II reference predictions for flights 1-25 are sourced from:

- *RASAero II Comparisons with Altitude Data* (Charles E. Rogers), <https://www.rasaero.com/comparisons-alt.htm>
- *RASAero II Comparison with MESOS 293K Flight Data, Rev. B* (Rogers), <https://www.rasaero.com/dloads/RASAero%20II%20Comparison%20with%20MESOS%20293K%20Flight%20Data%20-%20Rev%20B.pdf>

Flight 26 (Black Brant V VB, AAF-VB-32) is sourced from:

- *Black Brant Operations Report — AAF-VB-32, Churchill 3 March 1971*, NRC Canada / Space Research Facilities Branch, October 1971. DTIC accession AD0733141.

The corresponding RASAero `.CDX1` vehicle definition files for flights 1-25 are obtainable from a standard RASAero II installation and are **not redistributed** in this repository. The MESOS 293K vehicle file was provided by Rogers and is similarly not redistributed. The OpenRocket Plus model for flight 26 is auto-generated from a YAML audit of the source PDF; the build script and audit YAML live in the OpenRocket Plus repository at `paper/data/ork/sounding_rockets/` and trace every dimension, mass, and motor parameter to a specific page or table of AD0733141.

This repository contains only the matched-comparison artifact derived from those sources, plus the OpenRocket Plus predictions. To reproduce any row, see [`docs/reproducibility.md`](docs/reproducibility.md).

**Methodology note.** A subset of Rogers' RASAero II apogees (notably MESOS 293K and AeroPac 104K) are *post-flight* simulations in which the sustainer ignition delay and launch angle have been adjusted to match observed downrange distance at apogee. This is a standard reconstruction practice and is documented per-flight in Rogers' source PDF. OpenRocket Plus predictions in this database are pre-flight simulations from the unmodified `.CDX1` files; the comparison may slightly favor RASAero II for the small number of flights where post-flight reconstruction was performed.

## License

The matched-comparison artifact, the OpenRocket Plus predictions, and the curation in this repository are released under the **Creative Commons Attribution 4.0 International License** (CC-BY-4.0). See [LICENSE](LICENSE).

The underlying flight records and RASAero II reference predictions remain the work of Charles E. Rogers and the original modelers, and are referenced rather than republished here. When using this database, cite both the database and the underlying source (see Citation).

## Citation

Cite the database via its Zenodo DOI:

> Yu, A. (2026). *Rocket Flight Database* (v1.2) [Data set]. Zenodo. https://doi.org/10.5281/zenodo.19976138

A `CITATION.cff` is included for automatic citation tooling. The DOI above is the *concept* DOI for the latest release; specific versions have their own version-pinned DOIs available from the Zenodo record.

Always also cite the underlying flight records:

> Rogers, C. E. *RASAero II Comparisons with Altitude Data*. https://www.rasaero.com/comparisons-alt.htm

## Reproducibility

Each row in [`flight_comparison.csv`](flight_comparison.csv) is independently reproducible from public artifacts. Step-by-step instructions, version pins, and per-flight notes are in [`docs/reproducibility.md`](docs/reproducibility.md).

In short:

1. Install RASAero II and locate the `.CDX1` file for the vehicle.
2. Build OpenRocket Plus at the tag recorded in [`docs/reproducibility.md`](docs/reproducibility.md).
3. Import the `.CDX1` file via OpenRocket Plus's RASAero import path.
4. Run the simulation and compare the apogee against `apogee_thiswork_ft`.

## Roadmap

This database is intended to grow over time. Planned additions include:

- More flights from Rogers' published comparison set, plus other open ground-truth sources as they become available
- Altimeter make and model per flight (e.g., Featherweight Raven, RRC3)
- Launch site identifier and date
- Modeler attribution where available
- Additional simulator columns (other open-source or freeware low-fidelity codes)
- A `flights/` directory with per-flight JSON metadata if redistribution rights for vehicle files are obtained

Versioning follows [SemVer](https://semver.org). Schema-breaking changes bump the major version; new flights or columns bump the minor version.

## Contributing

Issues and pull requests welcome. To submit a new flight, open an issue with: the ground-truth source for the measured apogee, predicted apogees from the simulators of interest, and enough provenance information to reproduce the row.

## Contact

Aidan Yu — aidansyu@gmail.com
