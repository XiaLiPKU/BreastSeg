# Conditional GAN is All You Need for MR2CT

**Authors:** Xia Li, Muheng Li, Damien C. Weber, Antony J. Lomax, Joachim M. Buhmann, Ye Zhang
**Affiliations:** Paul Scherrer Institute, ETH Zurich, University Hospital of Zurich, University of Bern

---

## Motivation and Objectives

*   **Goal:** MR-based CT synthesis using conditional GANs (cGAN).
*   **Comparison:** Unbiased evaluation of 2D, 2.5D, and 3D networks under consistent conditions.
*   **Challenge:** Data augmentation for medical images, especially in 2D networks, is challenging. Traditional methods either segment along the z-axis and augment sections or augment the whole volume.
*   **Proposal:** An innovative **sampling plane augmentation** approach.

---

## Methods: Data Augmentation Strategies

The study compares three specific augmentation strategies for medical DL imaging:

1.  **Augment on 2D Slices (Traditional):**
    *   **Process:** Volume $\rightarrow$ Slicing $\rightarrow$ Randomly Resizing $\rightarrow$ Randomly Rotating.
    *   **Pros/Cons:** Straightforward to implement, but augmentation is limited to individual slices.

2.  **Augment on 3D Volume (True 3D):**
    *   **Process:** Volume $\rightarrow$ Randomly Resizing $\rightarrow$ Randomly Rotating $\rightarrow$ Slicing.
    *   **Pros/Cons:** Comprehensive, but applies uniform augmentation across all slices within a volume, compromising the element of randomness.

3.  **Augment on 2D Sampling Planes (Proposed):**
    *   **Process:** Volume + Centered Sampling Plane $\rightarrow$ Randomly Resizing (the plane) $\rightarrow$ Randomly Rotating (the plane) $\rightarrow$ Sampling from the Volume.
    *   **Pros/Cons:** Seeks to integrate the benefits of both strategies by focusing on the sampling plane rather than the sampled slices or the fixed volume.

---

## Key Results

*   **Dataset:** Separate networks for brain and pelvis. Split 150/30 (training/validation).
*   **Training:** 100 epochs.

### 1. Evaluation of Augmentation Parameters (Brain)

The table below shows how different rotation and resizing parameters affect the Mean Absolute Error (MAE) and Peak Signal-to-Noise Ratio (PSNR).

| Net width | Rotation | Resizing | MAE | PSNR |
| :--- | :--- | :--- | :--- | :--- |
| Thin | [0, 0] | [0, 0] | 82.03 | 23.20 |
| Thin | [-5, 5] | [-30, 30] | 80.94 | 24.33 |
| Thin | [-10, 10] | [-30, 30] | 80.75 | 24.35 |
| Thin | [-20, 20] | [-30, 30] | 80.19 | 24.41 |
| Thin | [-30, 30] | [-30, 30] | 79.70 | 24.45 |
| Thin | [-30, 30] | [-5, 5] | 81.10 | 24.34 |
| Thin | [-30, 30] | [-10, 10] | 82.52 | 24.24 |
| Thin | [-30, 30] | [-20, 20] | 80.03 | 24.47 |
| **Thin** | **[-30, 30]** | **[-30, 30]** | **79.07** | **24.46** |
| Wide | [-30, 30] | [-30, 30] | 77.94 | 24.60 |

### 2. Evaluation of Input Dimension and Shapes

Comparison between 2D, 2.5D, and 3D networks.

| Dim | Batch size | Input size | MAE | PSNR |
| :--- | :--- | :--- | :--- | :--- |
| 2D | 64 | 256 * 256 | 72.15 | 25.32 |
| 2D | 64 | 128 * 128 | 74.08 | 25.20 |
| **2.5D** | **16** | **4 * 128 * 128** | **67.58** | **25.90** |
| 2.5D | 8 | 8 * 128 * 128 | 69.32 | 25.72 |
| 2.5D | 4 | 16 * 128 * 128 | 69.69 | 25.71 |
| 3D | 16 | 16 * 64 * 64 | 70.58 | 25.66 |
| 3D | 16 | 32 * 32 * 32 | 72.97 | 25.47 |

---

## Conclusion

1.  **Network Performance:** The **2.5D network** demonstrates superior performance for the conditional GAN in MR2CT synthesis, surpassing both 2D and 3D networks (lowest MAE 67.58, highest PSNR 25.90).
2.  **Augmentation:** Data augmentation is imperative. The novel technique focusing on the **sampling plane** (as opposed to individual sampled slices) was validated as effective.
3.  **General:** Convolutional networks continue to be effective for this task.