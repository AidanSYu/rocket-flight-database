# Reproducibility Guide

Each row of [`flight_comparison.csv`](../flight_comparison.csv) is independently reproducible from public artifacts. This document walks through the process for any flight in the database, current or future.

## Prerequisites

1. **RASAero II.** Download and install from <https://www.rasaero.com/>. The bundled `Sample Files\` directory and the SimVReal flight set distributed with the installer contain the `.CDX1` vehicle definition files for most flights in the database. The MESOS 293K vehicle file is available directly from Charles E. Rogers on request.
2. **OpenRocket Plus.** Build from source: <https://github.com/AidanSYu/openrocketsupersonic>. Each release of this database is generated with a specific OpenRocket Plus tag, recorded below; using a different tag may produce ≤1% drift in `apogee_thiswork_ft`.

## Tag and version pinning

Pinned per database release. The values for the current release are:

| Component | Version (database v1.0) |
|---|---|
| RASAero II | as distributed by Rogers, no version override |
| OpenRocket Plus | branch `supersonic-aero-dev`, commit hash recorded in the release notes |
| JDK | 17 LTS (Temurin or Adoptium tested) |
| Operating system | Windows 11 (Java is portable; Linux/macOS should produce identical results to within floating-point determinism) |

Future database releases will record their pinned versions in the same row format.

## Reproducing a single row

For each flight, take the row in [`flight_comparison.csv`](../flight_comparison.csv) and follow these steps.

### Step 1: Locate the `.CDX1` file

The `vehicle_name` column of the CSV uses the names from the underlying source flight set. The matching `.CDX1` files in a default RASAero II install are typically under:

```
<RASAero II install dir>\Sample Files\
```

File names usually match the vehicle name with minor abbreviations (for example `Thunder&Lightning.CDX1`, `CalIsp1.CDX1`, `AeroPac104KStageOne&Two-2.CDX1`). Specialty files such as `MESOS 293K Flight.CDX1` are supplied separately by the source maintainer and are noted in the per-flight notes table below.

### Step 2: Verify the RASAero II prediction (optional)

Open the `.CDX1` file in RASAero II and run the standard flight simulation. The apogee altitude reported by RASAero II should agree with `apogee_rasaero_ft` within a few feet. Run-to-run drift of 50–500 ft has been observed for a small number of flights between RASAero II versions; treat the published Rogers values (`apogee_rasaero_ft` in this CSV) as authoritative.

### Step 3: Import into OpenRocket Plus

In OpenRocket Plus:

1. **File → Import → RASAero II (.CDX1)**, select the `.CDX1` file from Step 1.
2. Accept all defaults during import. Do not modify the imported geometry, motor, or launch conditions.
3. Verify the launch site altitude in the imported simulation matches the `launch_site_alt_ft` column.

### Step 4: Run the simulation

1. Select the primary simulation imported from the `.CDX1` file (typically `Simulation 1`).
2. Click **Run Simulation**.
3. Read the apogee from the simulation summary panel.

### Step 5: Compare

The reported apogee should match `apogee_thiswork_ft` to within ≤1% absolute drift. Larger discrepancies typically indicate either (a) a different OpenRocket Plus commit hash, or (b) a stale `.CDX1` file with a different motor or mass-properties entry than what Rogers' published comparison used.

## Per-flight notes

This table captures methodology nuances, file-name conventions, and known caveats per flight. New flights added in future releases append rows here.

| Flight | Notes |
|---|---|
| MESOS 293K | Two-stage (booster O4374, sustainer M787). RASAero II value is a post-flight reconstruction with adjusted ignition delay and launch angle to match GPS downrange (per Rogers' Rev. B PDF). |
| AeroPac 104K Two-Stage | Two-stage (N1048 / M685W). RASAero II value uses launch-angle reconstruction to match GPS downrange. |
| ThreeCarbYen 2018 | Three-stage. Not currently in the database; candidate for a future release. |
| Caliber Isp 05 ARO-414 (Discovery / Columbia) | Two flights of the same vehicle. The source table does not distinguish them by name; the database uses team-assigned names from the modeler. |
| Caliber Isp 04 AVTC Team 1/2/3 | Three independent vehicles flown at the AVTC competition. |
| Proteus 6 | RASAero II prediction of 86,799 ft (Rogers' published value) is authoritative; earlier internal-only runs produced ≈81,500 ft due to RASAero II run-to-run drift. |

## Troubleshooting

**Q: My apogee differs by more than 1% from the published `apogee_thiswork_ft`.**
Check, in order: (1) you imported the same `.CDX1` file referenced in Rogers' published flight set, (2) you are on the right OpenRocket Plus commit, (3) the launch site altitude was preserved during import. The most common silent error is the launch site altitude defaulting to 0 ft on import for flights launched at high-altitude sites (Black Rock at ~3,933 ft, etc.).

**Q: The `.CDX1` file imports but the rocket has unusual stability or fin geometry warnings.**
Some legacy `.CDX1` files use construction options that map ambiguously to OpenRocket components (notably PodSet stacking and tube-fin transitions). The OpenRocket Plus importer logs warnings; check the import log. None of the flights in the current release require manual geometry intervention to reproduce the published apogees.

**Q: RASAero II gives me a different apogee than `apogee_rasaero_ft` for a particular flight.**
Run-to-run drift between RASAero II versions has been observed for several flights, particularly Proteus 6, Qu8k, Rabia, and Gibb. Use the values published by the upstream source (column `apogee_rasaero_ft`) as the authoritative reference.

## Reporting reproduction problems

Open an issue at <https://github.com/AidanSYu/rocket-flight-database/issues> with:

- The flight ID and vehicle name.
- The OpenRocket Plus commit hash you tested against.
- The apogee you got, and the absolute / relative difference from `apogee_thiswork_ft`.
- Any warnings from the RASAero II importer.
