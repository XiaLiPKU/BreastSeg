---
name: medtrans-source-code-overview
description: Treat MedTrans as the source of truth for method implementation details. Use this skill when writing Methods/Experiments sections or answering implementation-detail questions.
---

# MedTrans Source Code Overview

## Goal
Use `MedTrans/` as the canonical implementation source for method details in this project.
When a writing task needs exact algorithm behavior, parameter settings, training targets, ODE flow, or model structure, read the mapped code files first and describe what the code actually does.

## Scope
Applies to tasks that need method-level accuracy, including:
- writing `Methods` or `Experiments` text
- validating claims about implementation details
- locating equations and algorithm components in code
- checking config-driven hyperparameters

## Rules (Critical)
1. **Code is source of truth**: if a manuscript sentence conflicts with code, trust code.
2. **Read before writing**: for each subsection, read the mapped files first.
3. **Prefer concrete symbols**: cite function/class names when explaining behavior.
4. **Use primary config first**: start from `configs/ABCT_4/unet3D_b32m2_p64*64*64_3d_latent.yaml`, then follow `base:` links only if needed.

## Directory Structure (Annotated)
The tree below excludes `.git/`, `.vscode/`, and cache files.

```text
MedTrans/
├── train.py                                  # latent flow training entry
├── test.py                                   # latent flow inference entry
├── seg_train.py                              # semseg training entry
├── seg_test.py                               # semseg inference entry
├── rec.py                                    # reconstruction-related entry script
├── pytest.ini                                # pytest config
├── .gitignore                                # repo ignore rules
├── config_maisi_autoencoder.json             # MAISI autoencoder config
│
├── datasets/
│   ├── __init__.py                           # dataset package init
│   ├── readers.py                            # NIfTI readers and modality loading
│   ├── utils.py                              # augmentation coords, ROI offsets, sampling utils
│   └── dataset.py                            # training/test dataset logic and patch assembly
│
├── models/
│   ├── __init__.py                           # model package init
│   ├── layers.py                             # time embedding and reusable layers
│   ├── unet3d.py                             # baseline 3D UNet with timestep conditioning
│   ├── unet3d_post.py                        # post variant UNet implementation
│   └── vae.py                                # VAE wrappers and unified latent-flow model
│
├── flowing/
│   ├── __init__.py                           # flowing package init
│   ├── interpolater.py                       # rectified-flow training/interpolation logic
│   ├── sampler.py                            # ODE sampling/decoding path for inference
│   ├── semseg_trainer.py                     # semseg-specific trainer
│   ├── semseg_tester.py                      # semseg-specific tester
│   └── reconstructer.py                      # reconstruction helper routines
│
├── utils/
│   ├── __init__.py                           # utils package init
│   ├── utils.py                              # YAML merge, metrics, misc utility functions
│   ├── ssim.py                               # SSIM-related implementation
│   └── log_buffer.py                         # logging buffer utilities
│
├── configs/
│   ├── _base/
│   │   ├── datasets/
│   │   │   └── ABCT_4.yaml                   # dataset root/range/aug base config
│   │   ├── networks/
│   │   │   └── unet.yaml                     # base network config
│   │   └── schedules/
│   │       └── b16.yaml                      # base schedule/training config
│   └── ABCT_4/
│       ├── unet3D_b8_p64*64*64_3d_latent.yaml
│       ├── unet3D_b16_p64*64*64_3d_latent.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent.yaml           # primary experiment config
│       ├── unet3D_b32m2_p64*64*64_3d_latent_high_mask.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_within_mask.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_lr1e-3.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_lr1e-4.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_lr5e-4.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_lr5e-5.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_poly.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_pred_lbl.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_pred_lbl_adj.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_pred_lbl_poly.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_mvae.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_latent_mvae_pred_lbl.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_semseg.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_semseg_pred_lbl.yaml
│       ├── unet3D_b32m2_p64*64*64_3d_semseg_pred_lbl_poly.yaml
│       ├── unet3D_b64_p64*64*64_3d_latent.yaml
│       └── unet3D_b8m2_p96*96*96_3d_latent.yaml
│
├── scripts/
│   ├── run1gpu.sh                             # single-GPU train launch
│   ├── run2gpus.sh                            # multi-GPU train launch
│   ├── val1gpu.sh                             # single-GPU validation launch
│   ├── seg2gpus.sh                            # semseg multi-GPU launch
│   ├── seg_val1gpu.sh                         # semseg validation launch
│   ├── seg_launch2.sh                         # semseg launcher wrapper
│   ├── launch1.sh                             # launch wrapper 1
│   ├── launch2.sh                             # launch wrapper 2
│   ├── rec1gpu.sh                             # reconstruction run script
│   ├── debug.sh                               # debug run helper
│   ├── deepspeed.yaml                         # deepspeed runtime config
│   ├── inference.yaml                         # inference script config
│   ├── post_eval.py                           # post-evaluation utilities
│   ├── nii2npy.py                             # data conversion utility
│   └── format.sh                              # formatting helper script
│
└── tests/
    ├── test_vae.py                            # VAE unit tests
    ├── test_vae_integration.py                # VAE integration tests
    └── test_datasets/
        ├── test_config.yaml                   # dataset test config
        ├── test_dataset.py                    # dataset behavior tests
        └── test_utils.py                      # dataset utility tests
```

## Module Summary
- `datasets/`: data reading, ROI-aware coordinate sampling, train/test patch extraction.
- `models/`: latent model backbone (UNet variants) and MAISI VAE integration.
- `flowing/`: flow training targets, ODE-based sampling, and segmentation-specific trainers/testers.
- `configs/`: hierarchical experiment definitions with base inheritance.
- `utils/`: metrics and shared infrastructure (`load_yaml`, logging, SSIM, helpers).
- `scripts/`: standardized run/validation/post-eval launch helpers.
- `tests/`: sanity checks for key dataset and VAE components.

## Paper Section -> Source Mapping
When writing section text, read these files first.

### 2.1 Physics-Informed ROI Proposal
- `MedTrans/datasets/readers.py`
- `MedTrans/datasets/utils.py` (ROI offset and coordinate sampling helpers)
- `MedTrans/datasets/dataset.py` (bbox usage in data pipeline)

### 2.2 Sampling Plane Augmentation
- `MedTrans/datasets/utils.py`:
  - `get_aug_coords()`
  - `sample_coords()`
  - `cal_offsets()`
- `MedTrans/datasets/dataset.py`:
  - `_train_getitem()` (uses sampled coordinates + interpolation)

### 2.3 Rectified Flow for Segmentation
- `MedTrans/flowing/interpolater.py` (training objective, target choices, mask strategy)
- `MedTrans/flowing/sampler.py` (ODE solve, latent trajectory, decoding path)
- `MedTrans/models/unet3d.py` (time-conditioned backbone design)
- `MedTrans/models/layers.py` (time embedding utilities)

### VAE Bottleneck Analysis
- `MedTrans/models/vae.py` (VAE wrapper and latent-space interface)
- `MedTrans/config_maisi_autoencoder.json` (autoencoder setup)

### Experiment Settings and Hyperparameters
- Primary: `MedTrans/configs/ABCT_4/unet3D_b32m2_p64*64*64_3d_latent.yaml`
- Bases:
  - `MedTrans/configs/_base/datasets/ABCT_4.yaml`
  - `MedTrans/configs/_base/networks/unet.yaml`
  - `MedTrans/configs/_base/schedules/b16.yaml`
- Merge logic: `MedTrans/utils/utils.py` (`load_yaml()`)

## Entry Points
Start here when tracing end-to-end behavior:
- `MedTrans/train.py` -> latent flow trainer path
- `MedTrans/test.py` -> latent flow inference path
- `MedTrans/seg_train.py` -> semseg trainer path
- `MedTrans/seg_test.py` -> semseg inference path

## Recommended Workflow
1. Identify manuscript subsection or technical question.
2. Jump to the mapped file set above (usually 2-4 files).
3. Confirm concrete symbols/parameters from code and config.
4. Only then draft narrative text with implementation-accurate wording.
