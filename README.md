# Rocket Flight Database

A public, maintained database of rocket flight ground-truth data with matched aerodynamic-simulator predictions, for model validation and benchmarking.

Each entry pairs a measured flight outcome (peak apogee from on-board instrumentation) with the corresponding predictions from one or more low-fidelity trajectory simulators on the as-flown vehicle. Researchers and tool authors can use the database to evaluate accuracy, identify systematic biases, and compare against alternative codes on identical inputs.

The database currently includes RASAero II and [OpenRocket Plus](https://github.com/AidanSYu/openrocketsupersonic) as the matched simulators. Additional simulators may be added in future releases.

## Current contents (v1.0)

The first release includes 25 flights spanning Mach 0.54 to Mach 4.33 and apogee altitudes from 3,577 ft to 293,488 ft, drawn from the public flight-comparison set published by Charles E. Rogers (RASAero II author).

Aggregate accuracy at v1.0:

| Predictor | Mean \|err\| | Within ±5% | Within ±10% |
|---|---:|---:|---:|
| RASAero II | 5.26% | 13 / 25 | 22 / 25 |
| OpenRocket Plus | 4.49% | 15 / 25 | 25 / 25 |

These aggregates apply to the v1.0 contents and will update with each release.

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
| `apogee_rasaero_ft` | RASAero II predicted apogee for the as-flown vehicle, feet |
| `apogee_thiswork_ft` | OpenRocket Plus predicted apogee for the as-flown vehicle, feet |
| `err_rasaero_pct` | Signed RASAero II error: 100 × (rasaero − real) / real |
| `err_thiswork_pct` | Signed OpenRocket Plus error: 100 × (thiswork − real) / real |
| `abs_err_delta_pp` | Absolute-error delta, percentage points: \|err_rasaero\| − \|err_thiswork\|. Positive means OpenRocket Plus is closer to the measured apogee on this flight |
| `flight_data_type` | Instrumentation that produced the measured apogee (Barometric Altimeter, GPS, Optical Track, Integrated Accelerometer) |
| `data_source` | Provenance pointer for the underlying flight record and reference prediction |

The schema may grow in future releases (altimeter make/model, launch site identifier, modeler attribution, additional simulator columns). Schema-breaking changes bump the major version.

## Source disclosure

Measured apogees, motor configurations, vehicle diameters, and RASAero II reference predictions are sourced from:

- *RASAero II Comparisons with Altitude Data* (Charles E. Rogers), <https://www.rasaero.com/comparisons-alt.htm>
- *RASAero II Comparison with MESOS 293K Flight Data, Rev. B* (Rogers), <https://www.rasaero.com/dloads/RASAero%20II%20Comparison%20with%20MESOS%20293K%20Flight%20Data%20-%20Rev%20B.pdf>

The corresponding RASAero `.CDX1` vehicle definition files are obtainable from a standard RASAero II installation and are **not redistributed** in this repository. The MESOS 293K vehicle file was provided by Rogers and is similarly not redistributed.

This repository contains only the matched-comparison artifact derived from those sources, plus the OpenRocket Plus predictions. To reproduce any row, see [`docs/reproducibility.md`](docs/reproducibility.md).

**Methodology note.** A subset of Rogers' RASAero II apogees (notably MESOS 293K and AeroPac 104K) are *post-flight* simulations in which the sustainer ignition delay and launch angle have been adjusted to match observed downrange distance at apogee. This is a standard reconstruction practice and is documented per-flight in Rogers' source PDF. OpenRocket Plus predictions in this database are pre-flight simulations from the unmodified `.CDX1` files; the comparison may slightly favor RASAero II for the small number of flights where post-flight reconstruction was performed.

## License

The matched-comparison artifact, the OpenRocket Plus predictions, and the curation in this repository are released under the **Creative Commons Attribution 4.0 International License** (CC-BY-4.0). See [LICENSE](LICENSE).

The underlying flight records and RASAero II reference predictions remain the work of Charles E. Rogers and the original modelers, and are referenced rather than republished here. When using this database, cite both the database and the underlying source (see Citation).

## Citation

Cite the database as:

> Yu, A. (2026). *Rocket Flight Database, v1.0*. https://github.com/AidanSYu/rocket-flight-database

A `CITATION.cff` is included for automatic citation tooling. A Zenodo DOI will be minted from a tagged release; once available, prefer the DOI over the GitHub URL.

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
