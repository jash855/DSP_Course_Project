# DSP_Course_Project
# Fire and Smoke Detection using Digital Signal Processing

A vision-based fire and smoke detection system implemented in Python using DSP techniques. The system performs pixel-level color segmentation, morphological filtering, and texture-based false positive rejection to detect fire and smoke regions in images with bounding box output.

---

## Overview

Traditional fire alarms rely on heat or smoke sensors placed at fixed ceiling points. This project takes a different approach — using camera images to detect fire and smoke visually, in the same way a human observer would. The system analyzes each pixel's color, filters noise, removes false positives through texture analysis, and draws bounding boxes around detected hazard regions.

Implemented in approximately 165 lines of Python. No OpenCV dependency. Runs fully inside Jupyter Notebook.

**Fire Detection Accuracy — 99.91%**  
**Smoke Detection Accuracy — 98.68%**

---

## Pipeline

The system processes images through a seven-stage pipeline:

**Stage 1 — Scene Generation**  
A 200x200 synthetic test image is created with a dark background, an orange-red fire region, a bright yellow flame tip, a gray smoke cloud, and a red wall patch to simulate a real-world false positive. Random noise is added to simulate camera grain.

**Stage 2 — Fire Color Detection**  
Each pixel is checked against four color rules derived from Chen et al. (IEEE ICIP, 2004):
- Red channel > 0.6
- Green channel > 0.2
- Blue channel < 0.35
- Red > Green > Blue (color dominance order)

Pixels satisfying all four rules are flagged as fire candidates.

**Stage 3 — Smoke Color Detection**  
Smoke appears gray in camera imagery. Pixels are flagged as smoke if the difference between R-G and R-B channels is each below 0.12, and brightness falls within a moderate range (0.3 to 0.65) — excluding dark backgrounds and bright fire regions.

**Stage 4 — Morphological Cleanup**  
Binary Opening (erosion followed by dilation) removes small isolated noise pixels. Binary Closing (dilation followed by erosion) fills small holes within detected blobs, producing clean, solid regions.

**Stage 5 — Texture-Based False Positive Rejection**  
Objects like red walls can survive morphological cleanup due to their size. A texture check resolves this: the system converts each detected region to grayscale and computes the variance of pixel intensities within it. Regions with variance below 0.003 are classified as smooth surfaces (walls, furniture) and discarded. Real fire has rough, irregular texture and passes this threshold. Based on Toreyin et al. (IEEE, 2006).

**Stage 6 — Bounding Box**  
The largest connected fire and smoke regions are located. A yellow rectangle is drawn around the fire region and a cyan rectangle around the smoke region.

**Stage 7 — Accuracy Evaluation**  
Since the scene is synthetically generated, exact ground truth masks are known. The detected mask is compared pixel-by-pixel against the true mask to compute a percentage accuracy score.

---

## Results

| Stage | Fire Pixels | Notes |
|---|---|---|
| After Color Detection (Raw) | 2,427 px | Fire region and red wall both flagged |
| After Morphological Cleanup | 2,427 px | Small noise removed, wall still present |
| After Texture Check | 2,427 px | Red wall removed, real fire kept |
| Fire Detection Accuracy | 99.91% | Compared against ground truth mask |
| Smoke Detection Accuracy | 98.68% | Compared against ground truth mask |

The texture check successfully eliminated the red wall false positive while preserving the actual fire region. Both hazards are detected at over 98% accuracy with correctly placed bounding boxes.

---

## Repository Structure

```
fire-smoke-detection-dsp/
|
|-- fire_updated.py          # Full Python source
|-- fire_detection.ipynb     # Jupyter Notebook version
|-- fire_detection.png       # Output image (pipeline stages)
|-- README.md
```

---

## Requirements

```
Python 3.8+
numpy
matplotlib
scipy
```

Install dependencies:

```bash
pip install numpy matplotlib scipy
```

---

## Usage

**Run the Python script directly:**

```bash
python fire_updated.py
```

**Or open the Jupyter Notebook:**

```bash
jupyter notebook fire_detection.ipynb
```

The script will generate the output image `fire_detection.png` showing all pipeline stages side by side.

---

## Limitations

- Tested on a synthetic scene only. Performance on real video footage may be lower due to complex lighting and varied backgrounds.
- Textured red surfaces such as brick walls or patterned orange wallpaper may pass the texture check and trigger a false detection.
- Gray fog, light-colored walls, or dust may be incorrectly flagged as smoke due to the broad gray color rule.
- The system processes single images, not video. Real fire flickers between frames — integrating motion detection across frames would reduce false alarm rates significantly.
- No thermal or heat sensor data is incorporated. Production systems typically fuse visual detection with heat sensing for higher confidence.
- Only the largest connected region is bounded. Multiple separate fire areas in a single image would produce only one bounding box.

---

## References

1. T. H. Chen, P. H. Wu, and Y. C. Chiou, "An Early Fire-Detection Method Based on Image Processing," IEEE International Conference on Image Processing (ICIP), 2004.

2. B. U. Toreyin, Y. Dedeoglu, and A. E. Cetin, "Computer Vision Based Method for Real-Time Fire and Flame Detection," Pattern Recognition Letters, vol. 27, no. 1, pp. 49–58, 2006.

3. R. C. Gonzalez and R. E. Woods, Digital Image Processing, 4th ed., Pearson / IEEE, 2018.

4. N. Otsu, "A Threshold Selection Method from Gray-Level Histograms," IEEE Transactions on Systems, Man, and Cybernetics, vol. 9, no. 1, pp. 62–66, 1979.

---

## License

This project is for academic purposes. See LICENSE for details.
