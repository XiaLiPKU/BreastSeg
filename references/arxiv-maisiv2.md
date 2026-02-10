# MAISI-v2: Accelerated 3D High-Resolution Medical Image Synthesis with Rectified Flow and Region-specific Contrastive Loss

**Authors:** Can Zhao, Pengfei Guo, Dong Yang, Yucheng Tang, Yufan He, Benjamin Simon, Mason Belue, Stephanie Harmon, Baris Turkbey, Daguang Xu
**Affiliations:** NVIDIA, National Institutes of Health, University of Oxford, University of Arkansas for Medical Sciences
**Venue:** arXiv:2508.05772v2 [cs.CV] 16 Nov 2025
**Code:** https://github.com/NVIDIA-Medtech/NV-Generate-CTMR/tree/main
**GUI Demo:** https://build.nvidia.com/nvidia/maisi

---

## Abstract

MAISI-v2 is a framework for accelerated and condition-consistent 3D medical image synthesis. It addresses limitations in existing diffusion models regarding generalizability, slow inference, and weak alignment with input conditions.
*   **Key Innovation:** Integrates **Rectified Flow** for fast generation and a novel **Region-specific Contrastive Loss** to enhance sensitivity to regions of interest (ROI) like small tumors.
*   **Performance:** Achieves SOTA image quality with **33× acceleration** compared to Latent Diffusion Models (LDM).
*   **Utility:** Synthetic images effectively improve downstream segmentation tasks via data augmentation.

---

## 1. Introduction

*   **Challenges in Medical Synthesis:**
    1.  **Generalizability:** Models often trained on narrow datasets (specific organs/modalities) fail on real-world clinical data.
    2.  **Inference Speed:** Diffusion models (DDPM) require hundreds of steps, making 3D high-res synthesis (e.g., $512^3$) computationally prohibitive.
    3.  **Condition Fidelity:** Weak alignment between conditioning inputs and outputs.
*   **MAISI-v2 Solution:** Builds on MAISI but replaces DDPM with Rectified Flow for speed and introduces contrastive loss for precise control.

---

## 2. Methodology

### 2.1 Overview
1.  **VAE:** Compresses 3D volumes (spatial compression $4 \times 4 \times 4$, 16x total).
2.  **Rectified Flow-based LDM:** Replaces DDPM to accelerate generation.
3.  **ControlNet:** Guides synthesis with region-specific contrastive losses.

### 2.2 Rectified Flow (Background)
Learns a straight path ODE between noise ($\pi_0$) and data ($\pi_1$).
ODE:
$$\frac{d\phi_t(x)}{dt} = v_t(\phi_t(x)), \quad \phi_0(x) = x_0, \quad \phi_1(x) = x_1$$

Training Objective ($x_t = (1-t)x_0 + tx_1$):
$$\mathcal{L}_{flow} = \int_0^1 \mathbb{E}_{x_0,x_1,t} \left[ \|v_t(x_t, c) - (x_1 - x_0)\|^2 \right] dt$$

### 2.3 Training Strategy (3 Stages)
1.  **Pre-training:** Low-res ($128^3$), large batch size (96), 1 day.
2.  **Main Stage:** Full-res, bucketed data parallelism (grouping by shape), 10 days.
3.  **Fine-Tuning:** Mixed shapes with sampling weights to address data imbalance, 10 days.

### 2.4 Region-specific Contrastive Loss
To enforce fidelity, the model generates two outputs from the same noise: one with the original mask ($c_{orig}$) and one with a perturbed mask ($c_{perturb}$) where ROI labels are swapped with background.

*   **ROI Sensitivity Loss ($\mathcal{L}_{roi}$):**
    $$D_{roi} = \|(G_\theta(x_t, c_{orig}) - G_\theta(x_t, c_{perturb})) \odot m\|_{1,m}$$
    $$\mathcal{L}_{roi} = -\min(D_{roi}, \delta)$$
    *(Upper bound $\delta=2$ prevents gradient explosion).*

*   **Background Consistency Loss ($\mathcal{L}_{bg}$):**
    $$m^- = 1 - \text{dilate}(m)$$
    $$\mathcal{L}_{bg} = \|(G_\theta(x_t, c_{orig}) - G_\theta(x_t, c_{perturb})) \odot m^-)\|_{1,m^-}$$

*   **Final Objective:**
    $$\mathcal{L}_{total} = \mathcal{L}_{flow} + \lambda_{contrast}(\mathcal{L}_{roi} + \mathcal{L}_{bg})$$

---

## 3. Experiments

### 3.1 Training Data
*   **Source:** 11k CT scans (107k images) from public datasets (MAISI, HNSCC, AMOS22, LIDC, etc.) and in-house datasets (Bone lesion, chest/abdomen).
*   **Resolution:** High-res volumes up to $512 \times 512 \times 768$.

### 3.2 Evaluation of Accelerated LDM

**Table 1: FID scores and single 80G GPU inference time on OOD dataset AutoPET2023**
*Images: $512 \times 512 \times 512$, $1mm^3$ spacing.*

| Model | Steps | Time (s) | FID$_{xy}$ | FID$_{yz}$ | FID$_{xz}$ | FID$_{avg}$ | Device |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **HA-GAN** | 1 | 1 | 13.813 | 12.567 | 14.405 | 13.595 | H100 |
| **MedSyn** (2-stage DDIM) | 50+20 | 100 | 18.662 | 22.171 | 33.293 | 24.709 | H100 |
| **GenerateCT** (2D EDM) | 25$\times$201 | 89 | 7.909 | 9.256 | 15.106 | 10.757 | H100 |
| **MAISI** (DDPM) | 1000 | 198+15 | 2.199 | 2.480 | 2.642 | 2.441 | H100 |
| **MAISI** (DDIM) | 30 | 6+15 | 4.855 | 4.703 | 4.770 | 4.776 | H100 |
| **MAISI-v2** (Rectified Flow)| **30** | **6+15** | **2.217** | **2.211** | **2.538** | **2.322** | **H100** |

*   **Result:** MAISI-v2 achieves comparable/better FID than MAISI (DDPM 1000 steps) but is significantly faster (21s vs 213s total).

**Table 2: Inference time comparison vs Video Models (50 steps)**

| Method | Output Size | # Voxels | Time (s) | Device |
| :--- | :--- | :--- | :--- | :--- |
| SVD (Video) | $3 \times 576 \times 1024 \times 25$ | 4.4E7 | 100 | A100 |
| Open Sora 2.0 (Video)| $3 \times 768 \times 768 \times 128$ | 2.3E8 | 162 | H200 |
| **MAISI-v2** (Uncond.)| $512 \times 512 \times 512$ | 1.3E8 | **26** | H100 |
| **MAISI-v2** (Cond.) | $512 \times 512 \times 512$ | 1.3E8 | **34** | H100 |

### 3.3 Ablation Studies

**Inference Steps:** Quality improves up to 30 steps, with diminishing returns after.

**Table 3: FID scores across inference steps for MAISI-v2 LDM**

| Metric | 5 | 10 | 20 | 30 | 40 | 50 | 70 | 100 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **FID$_{avg}$** | 20.334 | 4.421 | 2.645 | **2.322** | 2.204 | 2.064 | 1.981 | 1.967 |

**ControlNet & Downstream Segmentation:**
MAISI-v2 synthetic data improves segmentation accuracy (Dice score) when used for augmentation.

**Table 4: Segmentation Dice scores (Real + Synthetic)**
*Baselines: Real Only, DiffTumor, MAISI.*

| Method | Liver Tumor | Lung Tumor | Panc. Tumor | Colon Tumor | Bone-Les |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Real Only** | 0.662 | 0.581 | 0.433 | 0.449 | 0.504 |
| **DiffTumor** | 0.684 | - | 0.511 | - | - |
| **MAISI** | 0.688 | 0.635 | 0.482 | 0.485 | 0.539 |
| **ControlNet** (w/o Contra.)| 0.693 | 0.627 | 0.484 | 0.402 | 0.520 |
| **MAISI-v2** (w/ Contra.) | **0.695** | **0.655** | **0.497** | **0.491** | **0.537** |

*   MAISI-v2 outperforms baselines and ControlNet without contrastive loss in 4 out of 5 tasks.

---

## 4. Conclusion

MAISI-v2 utilizes Rectified Flow and new training losses to improve speed and condition accuracy for 3D medical image synthesis.
1.  **Speed:** 33x acceleration over DDPM.
2.  **Quality:** High-resolution generation ($512 \times 512 \times 768$).
3.  **Fidelity:** Region-specific contrastive loss ensures small lesions are accurately synthesized, proven by improved downstream segmentation performance.