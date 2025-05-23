
---

# Knee CT Scan Bone Segmentation Pipeline

[![Open in Kaggle](https://kaggle.com/static/images/open-in-kaggle.svg)](https://www.kaggle.com/code/aayushadhikari112/ra-selection-task-1)

## Submission Structure

All output files are inside the submission folder in the outputs directory of the Kaggle notebook link above:

- **Task1.1_BoneSegmentation**: Segmentation masks and previews
- **Task1.2_ContourExpansion**: Expanded masks (2mm, 4mm)  
- **Task1.3_RandomizedContour**: Randomized masks between original and expansion
- **Task1.4_LandmarkDetection**: All tibia masks and landmark coordinates

Each subfolder contains the files needed for reproducibility and evaluation.

---

## Project Overview

Hi! This repository walks you through my solution to segmenting bones in 3D knee CT scans—*no deep learning, just powerful, interpretable image processing.* This was developed for a research assistant task and is fully reproducible. Every step, from raw data to final anatomical landmarks, is documented and explained.

---

## Task Requirements

Here’s what the assignment called for:

1. **Bone Segmentation:** Identify and extract femur and tibia from a 3D CT scan using *only* image processing, and save the results as NIfTI (.nii.gz) masks.
2. **Contour Expansion:** Uniformly expand the segmented bone masks outward by a true metric distance (e.g., 2 mm).
3. **Randomized Contour Adjustment:** Create new masks that are randomized, but constrained to lie between the original and the expanded contours.
4. **Landmark Detection:** For the tibia, find and record the 3D coordinates of the medial and lateral lowest points for each mask variant.

We needed to produce and save five mask variations for the tibia:

* Original mask
* 2mm expanded mask
* 4mm expanded mask
* Randomized mask #1
* Randomized mask #2

---

## Why This Approach?

Instead of using deep learning (which needs tons of labeled data), I took a classic image processing approach, which actually works incredibly well here because:

* **Bone is high-contrast in CT:** Thresholding works!
* **CT geometry is consistent:** Anatomy is predictable, so spatial heuristics are reliable.
* **The pipeline is transparent and customizable:** Every step is visible, tweakable, and justifiable.
* **Easy to validate:** You can see exactly what’s happening at each stage—no “black box” magic.

This pipeline is perfect for situations where:

* Data is limited or labels are scarce.
* You need reproducible, interpretable steps.
* You want to guarantee anatomical validity, not just maximize some metric.

---

## Step-by-Step Pipeline: What, Why, and What You’ll See

---

### 1. Data Loading & Exploration

**What I do:**

* Load the CT scan and metadata (shape, voxel sizes).
* Plot an intensity histogram to see where bone and soft tissue “live”.

**Why:**

* It’s essential to confirm you’re working with the expected data type and range.
* The histogram tells you if intensity-based thresholding is going to be robust (hint: for bone, it is).

**Result:**

* Visual: Clear peaks for bone vs. soft tissue.
* Sanity check: CT image is loaded correctly and is in the right orientation.

---

### 2. Preprocessing

**What I do:**

* **Window** the intensity to \[-200, 2000 HU] to focus on bone and relevant tissue.
* **Denoise** with a Gaussian filter to smooth out noise but keep sharp edges.

**Why:**

* Windowing ignores irrelevant structures and outliers.
* Denoising makes thresholding and morphological ops work better.

**Result:**

* The bones are crisp and distinct in 2D slices.
* Histogram still shows clear bone peaks but with less noise.

---

### 3. Bone Segmentation

**What I do:**

* **Threshold** the scan using Otsu’s method (adaptive, data-driven).
* Apply **morphological closing/opening** and **hole filling** to clean up the mask.
* **Label** each 3D connected component (bone) and assign anatomical identities (femur, tibia, optionally fibula) based on size and spatial position.

**Why:**

* Adaptive thresholding ensures robustness across different scans.
* Morphological cleaning ensures contiguous, realistic bones.
* Labeling and spatial heuristics are crucial for identifying the right bones.

**Result:**

* Visual overlays: Each bone highlighted in a different color.
* Output: Femur and tibia are cleanly segmented, with no overlap.

---

### 4. Mask Refinement & Validation

**What I do:**

* Further refine each mask: more morphological cleaning, remove small specks/fragments.
* **Validate anatomical order:** If the “femur” is actually lower than the “tibia” in the scan, auto-swap them (correct anatomical labeling).

**Why:**

* Ensures only relevant, solid bone regions are kept.
* Guards against any orientation issues (e.g., if CT is flipped).

**Result:**

* 3D masks are now solid and anatomically correct.
* Visual overlays show perfect alignment with bones.

---

### 5. Contour Expansion (2mm, 4mm)

**What I do:**

* Expand each binary mask outward by a *true metric distance* (e.g., 2mm, then 4mm) using voxel sizes from the scan metadata.

**Why:**

* Expansion is needed in many clinical applications (implant fitting, margin analysis).
* Using actual voxel size ensures the expansion is accurate in millimeters, not just pixels.

**Result:**

* Visual: The expanded mask surrounds the original in every direction.
* Output: .nii.gz files for each expanded mask.

---

### 6. Randomized Contour Generation

**What I do:**

* For each original/expanded mask pair, I create a new mask by randomly selecting voxels from the “expansion band” (the region between the original and expanded mask).
* The process is parameterized and reproducible (random seed).
* Constraints: Never shrinks below original, never expands past the specified limit.

**Why:**

* These randomized masks simulate anatomical variability.
* Useful for stress-testing downstream analysis or machine learning.

**Result:**

* Visual: “Wobbly” but realistic mask variations, always within the original/expanded bounds.
* Output: Two unique randomized masks.

---

### 7. Landmark Detection

**What I do:**

* For the tibia, in each mask, I find the lowest (most inferior) axial slice with tibia.
* On that slice, I find the medial (leftmost) and lateral (rightmost) bone points.
* Convert voxel coordinates to real-world (mm) coordinates using the affine matrix.

**Why:**

* These landmarks are used for downstream anatomical analysis, validation, or registration.

**Result:**

* Output: A table of 3D coordinates (x, y, z) for each landmark in each mask, ready to submit.
* Visual: Optional overlays show landmark locations.

---

## Results

* **Masks** are saved as NIfTI files: original, 2mm, 4mm, randomized #1, randomized #2.
* **Landmark coordinates** are exported as a table (for all five tibial masks).
* **Intermediate overlays and histograms** provide clear QC at every step.

---

## How to Run This Project

1. **Install dependencies:**

   ```bash
   pip install nibabel scikit-image scipy numpy matplotlib
   ```

2. **Get the data:**
   Download the knee CT from [provided Google Drive link](https://drive.google.com/file/d/1NR7OEboARP_fpseIZOY0Wy8Ir1NEYfL5/view?usp=drive_link).

3. **Set your input path in the code:**

   ```python
   INPUT_PATH = '/path/to/your/knee_ct.nii.gz'
   ```

4. **Run the notebook or scripts:**

   * Either run the [Kaggle notebook](https://www.kaggle.com/code/aayushadhikari112/ra-selection-task-1) or
   * Clone this repo and run the notebook locally with Jupyter.

5. **Check your `results/` folder:**
   You’ll find all output masks and landmark coordinates, plus PNG visualizations for each major stage.

---

## Notes on Design Decisions

* All operations are fully 3D and respect physical spacing.
* The code is modular and readable: every major step is a function, and everything is logged for reproducibility.
* Output is compatible with 3D Slicer, ITK-SNAP, or any other medical imaging software.
* Randomized mask generation uses fixed seeds for reproducibility.

---

## Future Directions

Some ideas if you want to extend this:

* Benchmark this pipeline against deep learning methods (U-Net, V-Net, etc.).
* Add quantitative comparison to manual ground truth masks.
* Adapt the code for other joints (hip, ankle) or for MRI.
* Use the generated masks to automate cartilage thickness or joint space measurements.

---

## License

Available under the terms of the LICENSE file.

---

## Acknowledgments

Developed as part of a biomedical research assistant selection task. Data provided via public repository.

---

**Questions or feedback?**
Open an issue or drop me a message. I’m always happy to discuss anything!

---
