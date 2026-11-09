# Medical CT Analysis

Exploratory analysis of 3D CT scans, built as part of a team project.

> **Status:** work in progress. The repository currently covers data loading and
> exploration; modelling work will follow.

## What's here so far

- **Metadata exploration**: loads the dataset's `meta.csv` (patient age, gender,
  institute and study type, e.g. *ct polytrauma*, *ct thorax-neck*) with pandas.
- **Volume inspection**: loads CT volumes in NIfTI format (`ct.nii.gz`) with
  `nibabel` and checks:
  - volume shape and data type
  - anatomical orientation (RAS axis codes)
  - voxel spacing and size, and the image origin
  - mapping voxel indices to physical (world) coordinates through the affine
- **Intensity analysis**: the Hounsfield-unit range and a log-scaled intensity
  histogram for each scan.

## Project structure

```
.
├── main.ipynb         # exploration notebook
├── requirements.txt   # Python dependencies
├── COLAB_SETUP.md     # checklist for running on Google Colab with a GPU
└── dataset/           # local data (not committed, see below)
```

## Getting started

### Locally

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook main.ipynb
```

Put the data in `./dataset/`: one folder per scan (e.g. `dataset/s0011/ct.nii.gz`)
plus `dataset/meta.csv`.

### On Google Colab

See [`COLAB_SETUP.md`](COLAB_SETUP.md) for the session checklist: mounting Drive,
copying the data to local disk for faster I/O, and saving outputs before the
runtime disconnects.

## Data

The scans aren't included in this repository (`dataset/` is git-ignored)
because of their size and their medical nature.

## Tech stack

Python · Jupyter · NumPy · pandas · nibabel · Matplotlib
