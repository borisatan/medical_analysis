# Medical CT Analysis

A Python pipeline that takes 3D CT scans with femur segmentation masks, works
out which femurs are clean enough to use, and generates **digitally
reconstructed radiographs (DRRs)**, which are simulated 2D X-rays, from the CT
volumes.

Built as part of a team project.

> **Status:** work in progress. Data screening is done and DRR generation is
> working on single scans.

## Pipeline

### 1. Data exploration (`data_exploration.ipynb`)

Inspects individual scans before any processing: volume shape and data type,
anatomical orientation (RAS), voxel spacing, the affine that maps voxel indices
to real-world millimetre coordinates, and the Hounsfield-unit (HU) intensity
distribution.

[`INFO.md`](INFO.md) has worked notes on voxels and affine matrices, including
why a mask and its CT must share the same affine for voxel-level checks to be
valid.

### 2. Quality screening (`cleanData.ipynb`)

Each scan has a left and a right femur mask, and every mask goes through these
checks:

| Check | Why it matters | Rule |
| --- | --- | --- |
| Shape and affine match | The mask has to line up with the CT voxel for voxel | Same shape and same affine (`np.allclose`) |
| Orientation | Keeps left/right and front/back consistent | Orientation is RAS |
| Binary mask | Catches corrupted or soft-label masks | Values are only 0 and 1 |
| Femur present | Many scans (e.g. thorax-neck) don't cover the legs | Mask isn't empty |
| Femoral head complete | The scan can cut through the top of the bone | Mask doesn't touch the top slice |
| Enough bone | Needs enough length to be useful | At least 100 mm of femur |
| No metal implants | Hip replacements and screws distort HU values | Fewer than 50 voxels ≥ 2500 HU inside a slightly dilated mask |

It also flags scans where one femur is more than 25% larger than the other, for
manual review.

**Results.** 102 scans give 204 femur sides:

```
204 femur sides
 ├─ 61 excluded: femur not in scan
 ├─ 54 excluded: less than 100 mm of bone (6 of these also contained metal)
 ├─ 14 excluded: metal detected
 └─ 75 usable femur sides, from 47 scans
```

The results for each femur side (voxel count, volume in ml, extent, metal voxel
count, include flag and exclusion reason) are written to
[`outputs/femur_data.csv`](outputs/femur_data.csv).

### 3. DRR generation (`drr.ipynb`)

Turns the screened CT volumes into frontal 2D X-ray images:

1. Clip intensities to the range [-1000, 3000] HU.
2. Reorient to the canonical RAS orientation (`nib.as_closest_canonical`).
3. Convert HU to linear attenuation, μ = 0.02 · (1 + HU / 1000) mm⁻¹, using
   water ≈ 0.02 mm⁻¹ as the reference.
4. Sum μ along the front-to-back axis and multiply by the voxel spacing (a
   parallel-beam line integral), which gives a 2D radiograph.

## Project structure

```
.
├── data_exploration.ipynb   # single-scan inspection
├── cleanData.ipynb          # femur quality screening -> outputs/femur_data.csv
├── drr.ipynb                # DRR generation
├── INFO.md                  # notes on voxels and affines
├── COLAB_SETUP.md           # checklist for running on Google Colab
├── outputs/
│   └── femur_data.csv       # screening results for each femur side
├── requirements.txt
└── dataset/                 # local data (not committed, see below)
```

## Getting started

### Locally

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Expected data layout:

```
dataset/
├── meta.csv                          # scan metadata (semicolon-separated)
└── <scan_id>/
    ├── ct.nii.gz
    └── segmentations/
        ├── femur_left.nii.gz
        └── femur_right.nii.gz
```

### On Google Colab

See [`COLAB_SETUP.md`](COLAB_SETUP.md) for the session checklist: mounting Drive,
copying the data to local disk for faster I/O, and saving outputs before the
runtime disconnects.

## Data

The CT scans and masks aren't included in this repository (`dataset/` is
git-ignored) because of their size and their medical nature. The only data
committed is `outputs/femur_data.csv`, which holds measurements derived for each
femur.

## Tech stack

Python · Jupyter · NumPy · SciPy · pandas · nibabel · Matplotlib
