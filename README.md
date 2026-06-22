# Pt Nanoparticle Segmentation

An automated pipeline for detecting and segmenting noble-metal (Pt) nanoparticles in electron-microscopy images. It combines LoG candidate-center detection with Meta's **SAM (Segment Anything Model)** to produce particle-level segmentation and export statistics.

## Overview

Given a single STEM/TEM DM4 image, the notebook runs an end-to-end pipeline from preprocessing through particle segmentation, statistics, and visualization export:

```
Load DM4
  → separate sample / vacuum regions + normalize
  → CLAHE local contrast enhancement
  → region-based LoG candidate-center detection (coarse + residual passes)
  → SAM segmentation in three prompt modes (point / rescue / box)
  → union of the three results
  → particle statistics + export
```

## Files

| File | Description |
|------|-------------|
| `Pt_nanoparticle_segmentation.ipynb` | Main notebook with full code and step-by-step explanations |

## Dependencies

Python 3, ideally on a GPU-enabled machine (SAM inference is memory-hungry). Main dependencies:

```bash
pip install numpy scipy pandas matplotlib scikit-image hyperspy torch
pip install git+https://github.com/facebookresearch/segment-anything.git
```

| Library | Purpose |
|---------|---------|
| `hyperspy` | Read `.dm4` electron-microscopy data |
| `scikit-image` | CLAHE, LoG detection, watershed, morphology |
| `scipy` / `numpy` / `pandas` | Numerical computation and statistics |
| `torch` + `segment-anything` | SAM segmentation inference |
| `matplotlib` | Visualization |

> Before running, download the SAM model weights (e.g. `sam_vit_h_4b8939.pth`). The notebook's environment-setup cell handles the download/loading.

## Usage

1. Install the dependencies above.
2. Set the path to your `.dm4` image in the notebook's load cell.
3. Run the cells in order (0 Environment setup → 1 Preprocessing & candidate centers → 2 SAM segmentation & export).
4. If GPU memory is tight, run section 2.1 in batches using `START / END`, then merge the chunks with 2.1b.

## Pipeline Notes

- **Preprocessing**: separate the sample region from vacuum for normalization, then apply CLAHE for local contrast enhancement.
- **Candidate centers**: run LoG detection per region (thick / thin), plus a residual pass to recover missed particles.
- **SAM segmentation**: three prompt modes — point (batched, resumable), rescue (recover missed bright regions + watershed to split agglomerates), and box (boxes derived from particle size). Results are combined by union.
- **Output**: particle counts and size statistics, plus fill / boundary / solid visualizations.
