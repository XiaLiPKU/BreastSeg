# Fully automated breast segmentation on spiral breast computed tomography images

**Authors:** Sojin Shim, Davide Cester, Lisa Ruby, Christian Bluethgen, Magda Marcon, Nicole Berger, Jan Unkelbach, Andreas Boss
**Venue:** Journal of Applied Clinical Medical Physics (JACMP), 2022
**DOI:** https://doi.org/10.1002/acm2.13726

---

## Abstract

*   **Purpose:** To develop a fully automated method to segment breast components (adipose tissue, glandular tissue, skin, pectoralis muscle, skinfolds, ribs, and implants) on photon-counting Spiral Breast CT (SBCT) images for accurate breast density quantification.
*   **Method:** A framework consisting of four stages: (1) Component investigation/seed localization, (2) Segmentation of non-glandular components using adaptive seeded watershed, (3) Global breast density estimation based on HU, and (4) Iterative glandular tissue segmentation using adaptive seeded region growing.
*   **Results:** Evaluated on 25 datasets (distinct from 68 development datasets).
    *   **Segmentation Quality:** High agreement with manual segmentation (Dice Similarity Coefficient [DSC] ranges 0.90–0.97 for skin, muscle, and skinfolds).
    *   **Qualitative Score:** Radiologists rated the performance 4.7 $\pm$ 0.5 (on a 5-point Likert scale).
    *   **Density Bias:** Including mis-segmented skin or muscle in the breast volume resulted in a density error of 7.1% and 3.7%, respectively.
*   **Conclusion:** The proposed method enables accurate, fully automated quantification of breast density and glandular tissue, which is critical for breast cancer risk assessment.

---

## 1. Introduction

*   **Clinical Context:** Breast density is a strong risk factor for cancer and lowers mammography sensitivity (masking effect).
*   **Imaging Modality:** Spiral Breast CT (SBCT) with photon-counting detectors provides 3D, quantitative images (Hounsfield Units) without breast compression.
*   **Problem:** Accurate density assessment requires precise exclusion of skin, pectoralis muscle, and chest wall structures. Existing methods often rely on simple thresholding which is insufficient for SBCT due to overlapping HU values among soft tissues.

---

## 2. Methods

### 2.1 Dataset
*   **Scanner:** nu:view (AB-CT GmbH).
*   **Development Set:** 68 patients (used for HU calibration and parameter tuning).
*   **Test Set:** 25 representative scans (covering 4 BI-RADS density classes and implants).
*   **Target Components:** Adipose, Glandular, Skin, Pectoralis muscle, Skinfolds (thoracic/abdominal), Ribs, Implants.

### 2.2 Framework Overview
1.  **Preprocessing & Seed Detection:**
    *   Use Connected Component Analysis (CCA) to identify seeds for skinfolds, ribs, implants, and pectoralis muscle.
    *   Artifact removal (denoising) to distinguish compact glands from muscle.
2.  **Structural Segmentation (Non-Glandular):**
    *   **Algorithm:** Adaptive Seeded Watershed.
    *   **Logic:** Separates skin, muscle, and folds from the breast volume using gradient and distance markers derived from the seeds.
3.  **Iterative Glandular Segmentation:**
    *   **Global Estimation:** Calculate reference breast density using the mean HU of the "soft tissue" volume (excluding skin/muscle) via linear interpolation between pure fat and pure gland HU values.
    *   **Adaptive Region Growing:** Iteratively adjusts the threshold for the glandular seed until the segmented volume's percentage matches the global HU-derived density estimate. This handles partial volume effects better than fixed thresholding.

### 2.3 Evaluation
*   **Quantitative:** DSC, Difference Coefficient (DC) compared to manual segmentation by two radiologists.
*   **Qualitative:** 5-point Likert scale assessment by five experienced radiologists.
*   **Clinical Relevance:** Simulation of density estimation errors caused by mis-segmentation.

---

## 3. Results

**Table 2: Dice Similarity Coefficient (DSC) Results**
*Comparison between Automatic (C) and Manual Readers (A, B)*

| Component | Reader A vs B (Inter-rater) | Auto vs Readers (C vs AB) |
| :--- | :--- | :--- |
| **Overall** | 0.95 | 0.94 |
| **Skin** | 0.94 | 0.89 |
| **Pectoralis Muscle** | 0.94 | 0.94 |
| **Skinfold** | 0.96 | 0.99 |

*   The automatic method showed no significant difference from the inter-reader reproducibility.

**Qualitative Evaluation:**
*   Overall segmentation quality rated **4.7/5.0**.
*   Glandular tissue segmentation rated **4.5/5.0**.
*   Hard components (Ribs/Implants) rated near perfect (~4.9).

**Density Estimation Bias:**
*   Mean error between automated volumetric density and HU-derived density: **1.2%**.
*   **Impact of Errors:** If the skin is included in the breast volume (common in simple thresholding), density is overestimated by **7.1%**. If the pectoralis muscle is included, density is overestimated by **3.7%**.

**Computation Time:**
*   Automatic: ~13 mins per scan (21 mins with implants).
*   Manual: ~250 mins per scan.

---

## 4. Discussion

*   **Contribution:** First fully automated segmentation pipeline for SBCT that handles complex anatomy (muscles, skinfolds) and implants.
*   **Methodology:** Combines deterministic (morphological/watershed) and probabilistic (HU-based region growing) approaches.
*   **Clinical Value:**
    *   Accurate segmentation is prerequisite for reliable density scoring (breast cancer risk).
    *   Enables precise radiation dose calculation (organ dose).
    *   Can automate quality control (positioning check).
*   **Comparison:** Faster than MRI segmentation methods (often >20 min or requiring manual intervention).
*   **Limitations:** Deterministic steps may fail on extreme breast deformations (e.g., post-surgery); VAE/Deep Learning methods were not used but could be integrated in future work.