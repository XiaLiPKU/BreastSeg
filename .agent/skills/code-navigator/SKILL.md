---
name: medtrans-code-navigator
description: Navigate MedTrans code quickly by mapping common questions (augmentation, ROI sampling, training, inference, model architecture, configs, metrics) to exact Python and YAML files. Use when user asks about MedTrans implementation details or requests code changes under MedTrans.
---

# MedTrans Code Navigator

## Goal
Provide a stable file-routing map for `MedTrans/` so the agent can answer most questions with minimal reads, instead of repeatedly scanning the whole directory.

## Scope
Applies to all work under `MedTrans/`:
- data pipeline
- 3D patch augmentation
- latent flow / semseg training and inference
- model and VAE integration
- experiment configs

## Default Workflow (Important)
When asked about MedTrans internals:
1. Identify the question type from the routing table below.
2. Read only the listed 1-3 target files first.
3. Expand to neighbors only if the answer is incomplete.
4. Avoid broad directory reads unless explicitly requested.

## Quick Routing Table

### 1) "Augmentation is how implemented?"
- `MedTrans/datasets/utils.py`
  - `get_aug_coords()`: random scale/rotate/flip
  - `sample_coords()`: grid sampling coordinates
  - `cal_offsets()`: ROI-box-constrained random center offsets
- `MedTrans/datasets/dataset.py`
  - `_train_getitem()`: calls `sample_coords()`, then `F.grid_sample()`
- Main experiment config:
  - `MedTrans/configs/ABCT_4/unet3D_b32m2_p64*64*64_3d_latent.yaml`
    - `patch_size`, `scale_min/max`, `rotate_x/y/z`

### 2) "ROI sampling / patch extraction logic?"
- `MedTrans/datasets/dataset.py`
  - ROI bbox loading from `pkls/*.pkl`
  - train sampling vs test sliding-window logic
  - `assemble()` for patch-to-volume reconstruction
- `MedTrans/datasets/utils.py`
  - bbox-aware offset sampling and coordinate generation

### 3) "Data format and loading?"
- `MedTrans/datasets/readers.py`
  - reads `images/`, `masks/`, `cands/`, `tumors/` (`.nii.gz`)
- `MedTrans/configs/_base/datasets/ABCT_4.yaml`
  - `data_root`, intensity range, augmentation toggles

### 4) "Where does training start?"
- Latent flow training entry:
  - `MedTrans/train.py` -> `flowing/interpolater.py` (`Interpolater`)
- Semseg variant entry:
  - `MedTrans/seg_train.py` -> `flowing/semseg_trainer.py` (`SemSegTrainer`)

### 5) "Where does inference/test start?"
- Latent ODE sampling test:
  - `MedTrans/test.py` -> `flowing/sampler.py` (`Sampler`)
  - ODE path: `odeint(..., method="euler")`
- Semseg direct test:
  - `MedTrans/seg_test.py` -> `flowing/semseg_tester.py` (`SemSegTester`)

### 6) "Flow formulation and training target?"
- `MedTrans/flowing/interpolater.py`
  - latent interpolation over time
  - target choice: `pred_label` vs residual (`label_latent - image_latent`)
  - loss masking options: `loss_with_mask`, `highlight_mask`, `sample_adjust`
- `MedTrans/flowing/sampler.py`
  - latent ODE solve and decode back to voxel space

### 7) "Model architecture details?"
- `MedTrans/models/unet3d.py`
  - baseline 3D UNet with time embedding
- `MedTrans/models/unet3d_post.py`
  - post variant (`get_unet_3d_post`)
- `MedTrans/models/layers.py`
  - timestep blocks and embedding helpers

### 8) "VAE / latent bottleneck details?"
- `MedTrans/models/vae.py`
  - `MAISIVAE` wrapper, MAISI/MVAE selection
  - frozen VAE + trainable UNet in `UnifiedLatentFlowModel`
- `MedTrans/config_maisi_autoencoder.json`
  - MAISI autoencoder config stub

### 9) "How config inheritance works?"
- `MedTrans/utils/utils.py`
  - `load_yaml()` recursively merges `base:` configs
- Typical chain:
  - experiment YAML in `MedTrans/configs/ABCT_4/*.yaml`
  - `MedTrans/configs/_base/datasets/ABCT_4.yaml`
  - `MedTrans/configs/_base/networks/unet.yaml`
  - `MedTrans/configs/_base/schedules/b16.yaml`

### 10) "Metric definitions?"
- `MedTrans/utils/utils.py`
  - `seg_metrics()`: `mask_dice`, `soft_dice`, `cand_dice`, error metrics
  - `gen_metrics()` for reconstruction quality

### 11) "Run scripts and launch commands?"
- `MedTrans/scripts/run1gpu.sh`, `run2gpus.sh`
- `MedTrans/scripts/val1gpu.sh`, `seg_val1gpu.sh`
- `MedTrans/scripts/deepspeed.yaml`

### 12) "Need a sanity check before edits?"
- `MedTrans/tests/test_datasets/test_dataset.py`
- `MedTrans/tests/test_datasets/test_utils.py`
- `MedTrans/tests/test_vae.py`
- `MedTrans/tests/test_vae_integration.py`

## High-Value Configs (Read First)
If only one config is needed, prefer:
- `MedTrans/configs/ABCT_4/unet3D_b32m2_p64*64*64_3d_latent.yaml`

Then inspect bases only when required:
- `MedTrans/configs/_base/datasets/ABCT_4.yaml`
- `MedTrans/configs/_base/networks/unet.yaml`
- `MedTrans/configs/_base/schedules/b16.yaml`

## Known Project Terminology
Use consistent terms:
- "ROI-centered 3D patch augmentation"
- "random affine perturbations (scale/rotate/flip)"
- "latent flow matching"
- "MAISI VAE bottleneck"

Avoid reintroducing outdated wording such as "2D sampling plane augmentation" unless user explicitly asks about historical notes.

## Minimal-Read Policy
For most tasks, do not read beyond:
- 1 entry file (`train.py` / `test.py` / `seg_train.py` / `seg_test.py`)
- 1 implementation file (`flowing/*.py` or `datasets/*.py`)
- 1 config file (`configs/ABCT_4/*.yaml` plus base only if needed)

Only expand file reads if:
- symbol not found,
- behavior depends on inherited config,
- or user asks for deep audit/refactor.
