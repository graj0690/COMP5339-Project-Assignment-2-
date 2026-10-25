# Synthetic Dataset Generation

COMP5339 Assignment 2 — Task 1.

This package contains the code **and completed datasets** for 30 Assignment 1 DC
charger records over 1–14 September 2026. The supplied generator's model is used
with documented adaptations and real AEMO NSW wholesale prices. The full dataset
is in `data/final_dynamic/`; there is no need to generate it again before inspection.

## Results

| Measure | Delivered result |
|---|---:|
| Selected A1 charger records | 30: 19 urban-proxy, 11 regional-proxy |
| Assumed independent connectors | 103 |
| Hourly prices | 10,080 = 30 × 14 × 24 |
| Charging sessions | 4,007 |
| Session-start messages | 4,007 |
| Session-end messages | 4,007 |
| One-minute meter readings | 113,765 |
| Initial/changed-price messages | 6,360 |
| Combined publishing messages | 128,139 |
| Data validation | PASS, zero errors |
| Regression tests | 21 passed |

The rubric's “about 4,000 charging events” is interpreted here as **charging
sessions**: 4,007 sessions are 0.175% above 4,000. Session-start/end messages and
meter readings are counted separately, so this is not a claim of 4,000 total MQTT
messages. See `docs/RUBRIC_ALIGNMENT.md` for the interpretation and evidence.

## Use the included data

| File in `data/final_dynamic/` | Purpose |
|---|---|
| `stations.csv` | Selected charger attributes and simulated base tariffs |
| `prices_hourly.csv` | Complete hourly dynamic-price table |
| `sessions.csv` | Sessions, start/charging-end/unplug times, energy and bills |
| `session_metering.csv` | One-minute power/current/voltage/energy/SoC readings |
| `events.csv` | Globally ordered price/start/meter/end stream for Task 2 |
| `selected_station_crosswalk.csv` | Source identity, address, operator and SA4 for the 30 records |
| JSON reports | Source coverage, settings, sampling, hashes and validation |

`data/demo_dynamic/` contains a separately labelled **127-message** demonstration
with five complete sessions at two of the selected stations. At a 0.1-second gap,
the full stream takes at least 3.56 hours; the short demo takes at least 12.6 seconds.
Use a longer gap for a slower presentation. Read `docs/TASK2_HANDOFF.md` before
publishing either stream.

## Check or run the code

Use Python 3.9 or later; this package was generated and tested with Python 3.12.14.
No pip packages are required. Open a terminal in `synthetic_dataset_generation`.

Validate the included dataset:

```bash
python3 -m scripts.validate_outputs --data data/final_dynamic --aemo-cache data/aemo_cache
```

Run the tests:

```bash
python3 -m unittest discover -s tests -v
```

Generate another copy using the included AEMO cache:

```bash
python3 generate_synthetic_dataset.py --cache-only --out data/rerun_dynamic
```

Success ends with `Validation: PASS; errors=0`. Use a new output directory for
each run; existing generated CSVs are protected from overwrite. Running the main
script with no arguments writes to `data/generated`, leaving the included dataset
in `data/final_dynamic` intact. Allow at least 300 MB of free space for output and
temporary sorting. Exact seeded results depend on the same inputs and Python version.

The explicit command for the included run was:

```bash
python3 generate_synthetic_dataset.py --mode dynamic --start 2026-09-01 --days 14 --station-count 30 --selection-seed 5339 --seed 5339 --sessions-per-connector-day 3.434 2.576 --meter-every 1 --cache-only --out data/final_dynamic
```

The bundled `data/final_dynamic` already exists, so choose a different `--out`
when rerunning that command.

Preparation and selection only:

```bash
python3 generate_synthetic_dataset.py --mode prepare
```

For an updated A1 snapshot, use `--input path/to/updated_A1.csv`. Use
`--station-count 0` to simulate every valid DC candidate, or another positive count
for a different subset. These changes alter the dataset size and require a new run.
An actual complete, unique A1 database key can be supplied using `--id-column`;
the sparse source `OBJECTID` is not suitable. The adapter expects the documented A1
schema. Do not silently rename unrelated input fields to force a match.

## Selection and parameter choices

The A1 snapshot contains 1,951 records: 432 DC, 1,421 AC and 98 upcoming. Five DC
records with unresolved mixed ratings are documented and excluded, leaving 427
valid DC candidates. The supplied model describes DC fast charging, so AC and
upcoming records are outside the simulation scope.

Thirty candidates are sampled without replacement, proportionally across the
urban/regional proxy groups, using seed 5339. The selected records have 30 distinct
coordinate pairs. Source IDs and attributes are retained; there is no manually
chosen station list. `data/prepared/selection_report.json` records the exact sample
and its hashes. This is a reproducible modelling subset, not a claim of statistical
representativeness for all NSW charging activity.

Fourteen days include two complete weeks and produce 10,080 hourly rows. A single
pilot at the supplied arrival rates of 4/3 per connector/day produced 4,659 sessions.
Scaling those rates by 4,000/4,659 and rounding to three decimals gives **3.434 urban
and 2.576 regional**. The final fixed-seed run naturally produced 4,007 sessions;
none were trimmed, duplicated or added to force the count. This controls synthetic
data volume; it is not calibration against measured local demand.

The September 2026 NSW1 AEMO file is the recent complete-month wholesale source.
All 4,032 required five-minute intervals passed coverage checks. Twelve interval-
ending prices are averaged per hour and divided by 1,000 to convert AUD/MWh to
AUD/kWh. No missing observations are imputed.

The inherited hypothetical retail formula uses the station base tariff plus the
hourly wholesale deviation from the period mean, with a $0.10/kWh adder for
16:00 ≤ hour < 21:00, clipped to $0.20–$1.20/kWh. The supplied fleet, taper and
hourly demand profiles are retained; implementation changes are listed in
`docs/GENERATOR_CHANGES.md`. Timestamps use fixed AEST `+10:00`. The starting retail
tariff is locked for the session; occupancy lasts until unplug. One-minute meters
include terminal readings, including when a session finishes after the start window.

## Project files

- `generate_synthetic_dataset.py`: preparation, sampling, generation, events and QA.
- `generate_validated.py`: documented adaptation of the supplied model.
- `provided/`: original university code and documentation, preserved unchanged.
- `scripts/`: preparation, sampling, AEMO coverage, event ordering, validation and demo.
- `tests/`: generator and sampling regression tests; manufactured fixtures remain temporary.
- `data/assignment1/`: original source snapshot.
- `data/prepared/`: candidate list, selected list, crosswalks, exclusions and sampling audit.
- `data/aemo_cache/`: original downloaded September AEMO CSV.
- `data/final_dynamic/`: completed Task 1 dataset and reports.
- `data/demo_dynamic/`: short Task 2 presentation sample.
- `docs/`: rubric mapping, report material, schemas, changes, status and AI/source notes.
- `qa/`: actual run/test logs and volume-calibration evidence.

## GitHub and the team handoff

Extract the ZIP and upload the whole `synthetic_dataset_generation` folder,
preserving its subfolders. The complete final data is included. `.gitignore`
tracks this documented run while excluding later ad hoc output folders.

Task 2 receives `data/final_dynamic/` and `docs/TASK2_HANDOFF.md`. The group still
needs MQTT publication, the dashboard, DuckDB integration, its report and AI-use
declaration. Match the selected IDs to the actual A1 database using the supplied
crosswalk. Connector standards are absent from this snapshot and must come from
the team's augmented metadata. See `docs/STATUS.md` for remaining integration reviews.
