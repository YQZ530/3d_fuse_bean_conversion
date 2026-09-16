# 3D Fuse-Bead Instruction Converter

This repository converts photographed 3D fuse-bead instructions into one consolidated, printable pattern. Each instruction photo contains several intermediate build sections; the scripts extract those sections, recover their bead colors and shapes, pack them into one non-overlapping layout, and render a single large PNG/SVG/CSV pattern.

## Board width

The target board is **54 beads wide**. The generated pattern uses **52 active columns**, leaving one empty clearance column on the left and right. Existing output filenames therefore use `52x<height>` (for example, `sunflowers-52x59.png`).

If all 54 columns should contain beads instead of reserving the two edge columns, the current solver and renderer constants must be changed; the checked-in results were verified at 52 active columns.

## Repository layout

```text
.
|-- input/                 Original instruction photographs
|-- output/                Curated final PNG patterns
|-- results/               Per-photo extraction, layout, QA, and render artifacts
|   |-- 3648/
|   |-- ...
|   `-- 3656/
|-- scripts/               Extraction, packing, checking, and rendering scripts
|-- vendor/python-deps/    Legacy local dependency bundle (optional, git-ignored)
|-- requirements.txt       Python dependencies for a clean environment
`-- README.md
```

Run every command from the repository root because the scripts use repository-relative paths.

## Processing pipeline

For each source photo (`IMG_<id>.JPG`), the workflow is:

1. **Extract** — sample the photographed grids and save colors and part masks to `results/<id>/extracted.json`.
2. **Check** — create targeted color diagnostics for jobs that have a `check_<id>.py` script.
3. **Solve/pack** — place all extracted parts inside the 52-column active area without overlap and with at least one empty cell between separate parts.
4. **Build** — preserve project-specific placements and generate the matching renderer where required.
5. **Render** — create a printable PNG, scalable SVG, bead-code CSV, preview, and `verification.json`.
6. **Publish** — copy the preferred final PNG into `output/`.

The extraction coordinates are calibrated for the checked-in photos. A new photo or a differently cropped copy normally needs a new or adjusted extraction script.

## Setup

Python 3.11 or newer is recommended.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

The solver scripts also recognize the existing `vendor/python-deps` bundle. A virtual environment is preferable for new development.

## Re-render an existing pattern

The layout files are already checked in, so an existing result can be rendered without repeating extraction or optimization:

```powershell
python scripts/render_3656.py
```

This regenerates the Sunflowers artifacts under `results/3656/` and verifies the part count, bead count, dimensions, overlap, and clearance rules.

Two patterns also have an E22-background variant:

```powershell
python scripts/render_3652_e22.py
python scripts/render_3655_e22.py
```

E22 is a paper/background color only; it does not add beads or change the CSV bead grid.

## Run a complete existing job

The exact steps vary because some early jobs use hand-tuned placements. A typical solver-based job is:

```powershell
python scripts/extract_3656.py
python scripts/solve_3656.py
python scripts/build_3656.py
python scripts/render_3656.py
```

Job `3648` uses a specialized annealing/packing pipeline. Jobs `3649` and `3651` include hand-authored placement maps. Treat the scripts named with the same image ID as one job family.

## Output formats

- **PNG** — high-resolution printable pattern with grid and bead codes.
- **SVG** — scalable version of the same pattern.
- **CSV** — raw bead-code matrix; blank cells mean no bead.
- **JSON** — extracted parts, placements, and verification metadata.
- **Preview/verify images** — visual QA only, stored in `results/` rather than `output/`.

## Verified patterns

| Source | Pattern | Active grid | Parts | Beads |
|---|---|---:|---:|---:|
| 3648 | Starry Night | 52 x 80 | 37 | 2,428 |
| 3649 | Girl with a Pearl Earring | 52 x 60 | 13 | 1,919 |
| 3650 | A Thousand Miles of Mountains | 52 x 96 | 8 | 2,542 |
| 3651 | Flute Player | 52 x 64 | 22 | 1,843 |
| 3652 | Wheatfield with Cypresses | 52 x 90 | 18 | 2,906 |
| 3653 | The Scream | 52 x 83 | 14 | 2,295 |
| 3654 | Apple Bowler Figure | 52 x 59 | 11 | 1,914 |
| 3655 | The Great Wave | 52 x 80 | 33 | 2,504 |
| 3656 | Sunflowers | 52 x 59 | 19 | 2,040 |

The authoritative checks for each job are stored in `results/<id>/verification.json`.

## Adding another instruction photo

1. Put the original image in `input/` using a stable ID, such as `IMG_3657.JPG`.
2. Copy the closest `extract_<id>.py` script and recalibrate its regions, cell spacing, and palette reference points.
3. Add a matching solver/build/renderer set, keeping the active width at 52 columns for a 54-wide board with edge clearance.
4. Run the extraction and inspect all diagnostic images before accepting colors.
5. Solve and render the layout.
6. Confirm `verification.json`, visually inspect the final image, and copy the preferred PNG to `output/`.

Do not treat a successful render alone as proof of a correct transcription: color sampling and region boundaries still require visual review.
