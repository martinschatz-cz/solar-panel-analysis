# Datasets overview

## Introduction — solar panel imaging

Thermal and infra-red imaging are powerful, non-invasive tools for monitoring photovoltaic (PV) systems. Thermal (long-wave infrared) cameras measure surface temperature and reveal heat patterns such as hot-spots, diode failures, soiling and entire offline modules. Shorter-wavelength infrared cameras (near-/mid-IR) can also be useful for contrast enhancement and some defect types.

Why this matters:
- Detect faults early to reduce energy loss and maintenance costs.
- Localize and classify anomalies (hot-spots, cracks, soiling, shading).
- Enable automated inspection workflows using UAVs (drones) and deep learning.

Common uses for these datasets:
- Object detection / localization of PV panels in thermal frames.
- Anomaly classification at module or cell level.
- Segmentation and temperature-based analysis for condition monitoring.
- Domain adaptation and transfer learning between visible and thermal modalities.

---

## Dataset A — Thermal PV Panel Detection and Fault Detection (Zenodo)

- Link: https://zenodo.org/records/16420123
- DOI: https://doi.org/10.5281/zenodo.16420123
- Citation: Christakakis, P., Pechlivani, E.-M., & Dimou, P. (2025). Thermal PV Panel Detection and Fault Detection Dataset for UAV-Based Inspection [Data set]. Zenodo.

[![Zenodo](https://zenodo.org/badge/DOI/10.5281/zenodo.16420123.svg)](https://zenodo.org/record/16420123)

Overview:
- Modality: thermal imagery acquired from a DJI Mavic 3T UAV over a photovoltaic farm (Sindos, Thessaloniki).
- Processed images: a non-overlapping subset of roughly 350 images at 640×512 resolution containing full PV panel arrays.
- Typical uses: panel detection, fault (anomaly) detection, object counting and localization.

Provided annotations / splits (from dataset documentation):
- Training: 235 images — annotated PV panels: 18,487
- Validation: 83 images — annotated PV panels: 5,828
- Test: 35 images — annotated PV panels: 2,363
- Total annotated panels (approx.): 26,678

Notes:
- The dataset is especially well suited for object-detection experiments (detecting whole PV panels and counting panels per image).
- Image resolution and UAV capture geometry make this dataset appropriate for UAV-based inspection research and benchmarking.

---

## Dataset B — Infrared Solar Modules (Kaggle / Raptor Maps)

- Link: https://www.kaggle.com/datasets/marcosgabriel/infrared-solar-modules/data
- License: MIT (as published on Kaggle)
- Origin: provided by Raptor Maps; presented at ICLR 2020 (AI for Earth Sciences workshop)

Overview:
- Modality: cropped thermal / infrared images of individual photovoltaic modules.
- Task targets: anomaly classification across a taxonomy of module-level defects.
- Dataset structure: an `images/` folder containing cropped module images plus a `module_metadata.json` describing labels per image.

Classes (summary):
- Cell, Cell-Multi, Cracking, Hot-Spot, Hot-Spot-Multi, Shadowing, Diode, Diode-Multi, Vegetation, Soiling, Offline-Module, No-Anomaly

Notes:
- This dataset is targeted at classification of module-level anomalies and can be used for multi-class classification or per-class detection after localizing modules first.
- Because images are cropped to individual modules, this dataset is a good fit for patch-level CNNs and anomaly detectors.

---

## Quick comparison and suggested uses

- **Zenodo (Dataset A)**: best when you need full-frame UAV thermal imagery and object detection/counting of PV arrays.
- **Kaggle / Raptor Maps (Dataset B)**: best when you need labeled module crops for anomaly classification and patch-level model training.

Suggested experiments:
- Train an object detector on Dataset A to localize panels in UAV frames, then crop detections and classify with a model trained on Dataset B.
- Evaluate transfer learning between thermal and visible modalities if you have visible-band data.
