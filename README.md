# Facial Image Denoising on FER2013

## Overview

This project focuses on **denoising facial images** using learning-based image restoration. It uses the **FER2013** dataset as a source of 48×48 grayscale face images, but **does not perform recognition or classification**. Instead, the goal is to **reconstruct clean faces from corrupted inputs** and analyze reconstruction quality under different noise conditions.

---

## Project Description

1. **Problem Setting**  
   The task is formulated as single-image denoising: given a noisy face image \( \tilde{I} \), learn a mapping \( \mathcal{F}_\theta(\tilde{I}) \rightarrow \hat{I} \) that restores a clean estimate \( \hat{I} \) of the original image \( I \). The project evaluates how well a convolutional model can recover facial structure (edges, contours, expressions) from low-resolution, noise-corrupted inputs.

2. **Data**  
   - **Source:** FER2013 (48×48 grayscale facial crops).
   - **Usage:** Images are treated purely as **inputs for restoration**; their emotion labels are **not** used or predicted.
   - **Normalization:** Pixel values are scaled to a standard range to stabilize optimization.

3. **Noise Modeling**  
   To simulate real-world degradation, synthetic noise is applied to clean images before denoising:
   - **Salt-and-pepper noise** to mimic impulse corruption (random black/white pixels).
   - Optional **Gaussian noise** to represent sensor or low-light noise.
   - A small, fixed **noisy hold-out subset** may be reserved to test generalization of the learned denoiser.

4. **Modeling Approach**  
   The restoration model is a **convolutional autoencoder–style network**:
   - **Encoder:** stacked convolutional layers progressively compress features while preserving spatial context.
   - **Bottleneck:** compact representation that encourages the network to discard noise and retain facial structure.
   - **Decoder:** transposed/upsampling convolutions reconstruct the denoised image at the original 48×48 resolution.
   - **Regularization:** dropout or weight decay may be used to reduce overfitting; early stopping is driven by validation loss.

5. **Training Objective**  
   The model is trained to minimize a per-pixel **reconstruction loss** between prediction \( \hat{I} \) and clean target \( I \):
   - **Primary loss:** Mean Squared Error (MSE) or Mean Absolute Error (MAE).
   - The loss directly penalizes deviations in intensity, encouraging accurate reconstruction of facial details.

6. **Evaluation Methodology**  
   The project evaluates denoising quality using:
   - **Peak Signal-to-Noise Ratio (PSNR):** measures overall fidelity to the ground truth.
   - **Structural Similarity Index (SSIM):** assesses perceived structural preservation (edges, textures).
   - **Qualitative comparisons:** side-by-side visualizations of **noisy vs. denoised vs. clean** images to inspect restoration of eyes, nose, mouth contours, and hairline edges.

7. **Findings (Qualitative Summary)**  
   - The model consistently removes isolated impulse noise while **preserving key facial features**.
   - Over-smoothing can occur at high noise levels; fine textures (e.g., hair, subtle wrinkles) are the first to be lost when the model is tuned for aggressive denoising.
   - SSIM tracks perceptual quality better than PSNR for these low-resolution faces; increases in SSIM often correlate with clearer eye and mouth regions.

8. **Error Analysis**  
   - **High-contrast edges** (jawline, eyebrows) are denoised reliably; **low-contrast regions** (cheeks, forehead gradients) may exhibit slight banding or blurring.
   - **Dense impulse noise** leads to “filled-in” artifacts if the model is under-parameterized or early-stopped.
   - **Misaligned faces** or off-center crops are harder to reconstruct because the model learns strong spatial priors from typical face alignment within the dataset.
