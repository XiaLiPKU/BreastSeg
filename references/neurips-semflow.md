# SemFlow: Binding Semantic Segmentation and Image Synthesis via Rectified Flow

**Authors:** Chaoyang Wang, Xiangtai Li, Lu Qi, Henghui Ding, Yunhai Tong, Ming-Hsuan Yang
**Venue:** NeurIPS 2024
**Project Page:** https://wang-chaoyang.github.io/project/semflow

---

## Abstract

Semantic segmentation (SS) and semantic image synthesis (SIS) are modeled as a pair of reverse problems within a unified framework called **SemFlow**. Motivated by rectified flow theory, the authors train an ordinary differential equation (ODE) model to transport between the distributions of real images and semantic masks.
*   **Key Advantage:** The symmetric training objective allows reversible transfer between images and masks.
*   **For Segmentation:** Solves the contradiction between diffusion output randomness and segmentation uniqueness.
*   **For Synthesis:** A finite perturbation approach enhances diversity without changing semantic categories.
*   **Performance:** Competitive results on both tasks with fewer inference steps.

---

## 1. Introduction

*   **Problem:** SS and SIS are usually solved using distinct methodologies (Discriminative models for SS vs. GANs/Diffusion Models for SIS).
*   **Existing Diffusion Limitations in SS:**
    1.  Contradiction between random noise input and deterministic segmentation output.
    2.  High computational cost due to multiple inference steps.
    3.  Irreversibility between masks and images.
*   **SemFlow Solution:**
    *   Uses **Rectified Flow** to learn a direct mapping between images and masks (eliminating Gaussian noise as an intermediate).
    *   Models the process as an ODE with time-symmetric training, enabling bi-directional transmission.
    *   Introduces **Finite Perturbation** to enable one-to-many mapping for image synthesis.
    *   Straight transport trajectory reduces inference steps significantly.

---

## 3. Method

### 3.1 Diffusion Model and Segmentation Modeling (Background)

Standard diffusion forward process:
$$q(x_t|x_0) = \mathcal{N}(x_t; \sqrt{\alpha_t}x_0, (1 - \alpha_t)I), \quad t \in [0..T]$$

Training objective (L2 loss):
$$\mathcal{L} = \mathbb{E}_{x_0\sim q(x_0),\epsilon\sim\mathcal{N}(0,I),t} \left[ \left|\left| \epsilon_\theta(\sqrt{\alpha_t}x_0 + \sqrt{1 - \alpha_t}\epsilon_t) - \epsilon_t \right|\right|_2^2 \right]$$

Backward process (DDIM/DDPM unified):
$$x_{t-1} = \sqrt{\alpha_{t-1}} \left( \frac{x_t - \sqrt{1 - \alpha_t}\epsilon_\theta(x_t, t)}{\sqrt{\alpha_t}} \right) + \sqrt{1 - \alpha_{t-1} - \sigma^2_t} \epsilon_\theta(x_t, t) + \sigma_t\epsilon_t$$

### 3.2 Unify Segmentation and Synthesis with Rectified Flow

**Task-agnostic Framework:**
1.  **Pseudo Mask ($M$):** Semantic masks are converted to 3-channel pseudo masks to align with images $I$.
    $$m'_0 = \lfloor c/k^2 \rfloor, \quad m'_1 = \lfloor (c - m'_0 * k^2)/k \rfloor, \quad m'_2 = c - m'_0 * k^2 - m'_1 * k$$
    $$M = s * (m'_0, m'_1, m'_2)$$
    *(Where $c$ is category index, $k$ is division parts, $s$ is spacing).*
2.  **VAE Encoding:**
    $$z_0 = \mathcal{E}(I), \quad z_1 = \mathcal{E}(M), \quad \tilde{I} = \mathcal{D}(\tilde{z}_0), \quad \tilde{M} = \mathcal{D}(\tilde{z}_1)$$

**Bi-directional Training:**
Modeled using Rectified Flow where trajectory is $z_t = (1 - t)z_0 + tz_1$. The velocity field $v_\theta(z_t, t)$ is trained via:

$$\mathcal{L} = \int_0^1 \mathbb{E}_{(z_0,z_1)\sim\gamma} \left[ \left|\left| v_\theta(z_t, t) - (z_1 - z_0) \right|\right|^2 \right] \mathrm{d}t$$

The transfer is described by the ODE:
$$\frac{\mathrm{d}z_t}{\mathrm{d}t} = v_\theta(z_t, t), \quad t \in [0, 1]$$

**Inference:**
*   **Semantic Segmentation (Forward Flow, $z_0 \to z_1$):**
    $$z_{t+\frac{1}{N}} = z_t + \frac{1}{N}v_\theta(z_t, t)$$
*   **Semantic Image Synthesis (Reverse Flow, $z_1 \to z_0$):**
    $$z_{t-\frac{1}{N}} = z_t - \frac{1}{N}v_\theta(z_t, t)$$

### 3.3 Finite Perturbation

To enable multi-modal generation (one-to-many) and address low entropy of masks, amplitude-limited noise is added to the pseudo masks:

$$M' = M + \epsilon, \quad \epsilon \sim U(-\beta, \beta)$$

*   Condition: $0 < \beta < s/2$ ensures semantic labels do not change.
*   $z_1$ is replaced with $z'_1 = \mathcal{E}(M')$ during training and inference.

---

## 4. Experiments

### 4.1 Experimental Settings
*   **Datasets:** COCO-Stuff, CelebAMask-HQ, Cityscapes.
*   **Metrics:** mIoU (Segmentation); FID, LPIPS (Synthesis).
*   **Architecture:** Based on Stable Diffusion 1.5 UNet.

### 4.2 Main Results

**Table 1: Semantic segmentation and synthesis results**
*SS: Semantic Segmentation, SIS: Semantic Image Synthesis*

| Method | Task | Sampler | COCO-Stuff<br>mIoU (SS) | CelebAMask-HQ<br>FID (SIS) | CelebAMask-HQ<br>LPIPS (SIS) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **MaskFormer** | SS | - | 41.9 | - | - |
| **DSM (DDIM-200)** | SS | - | 16.1 | - | - |
| **DSM (DDPM-200)** | SS | - | 20.2 | - | - |
| **pix2pixHD** | SIS | - | - | 54.7 | 0.529 |
| **SPADE** | SIS | - | - | 42.2 | 0.487 |
| **SC-GAN** | SIS | - | - | 19.2 | 0.395 |
| **BBDM-200** | SIS | - | - | 21.4 | 0.370 |
| **SemFlow (Ours)** | **SS & SIS** | **Euler-25** | **38.6** | **32.6** | **0.393** |

*   **Segmentation:** SemFlow significantly outperforms Diffusion-based Segmentation Models (DSM) and approaches specialist discriminative models.
*   **Synthesis:** Achieves competitive FID and LPIPS scores compared to GANs and other diffusion methods.

### 4.3 Ablation Studies

**Straight Trajectory (Inference Steps):**
SemFlow maintains high performance even with very few inference steps compared to DSM.

**Table 2: Semantic segmentation results with different inference steps on COCO-Stuff (mIoU)**

| Method & Sampler | 1 | 2 | 5 | 10 | 25 | 50 | 100 | 200 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **DSM (DDIM)** | - | 0.1 | 4.0 | 9.5 | 13.6 | 14.9 | 15.7 | 16.1 |
| **DSM (DDPM)** | - | 0.1 | 5.3 | 12.3 | 17.5 | 18.9 | 19.6 | 20.2 |
| **SemFlow (Euler)**| **28.3**| **31.0**| **36.9**| **38.4**| **38.6**| **38.4**| **38.3**| **38.3**|

*   **Perturbation:** Without perturbation, the model generates uni-modal, over-smoothed results. With perturbation, it generates diverse, high-quality images.

---

## 5. Conclusion

SemFlow employs a diffusion model (via Rectified Flow) to integrate semantic segmentation and semantic image synthesis as a pair of reverse problems.
1.  **Reversible:** Models segmentation as a transport problem between image and mask distributions.
2.  **Diverse:** Finite perturbation enables multi-modal generation.
3.  **Efficient:** Straight trajectory modeling allows sampling with very few steps (e.g., competitive results at 1 step).