# Hybrid-SiamRPN-CSRDCF-Tracker

A hybrid visual object tracking framework that combines a pretrained DaSiamRPN tracker with a CSRDCF-style correlation filter (OpenCV CSRT) using learned, per-frame adaptive fusion weights. A supervised regression model predicts how much to trust each tracker on every frame, using features computed from the two trackers' own outputs.

This repository accompanies the Master's dissertation:

**Hybrid SiamRPN–CSRDCF Tracker for Robust Visual Object Tracking**
Hanaa Parvez Khan, Liverpool John Moores University, 2024.

> **Scope note.** All results below come from a four-sequence pilot subset of OTB2013 (Basketball, Bolt, Boy, Car4), each truncated to its first 100 frames. They are a pilot-scale trend, not a full-benchmark result. See [Limitations](#limitations).

---

## Overview

Visual object tracking remains challenging under occlusion, illumination variation, motion blur, scale variation, and background clutter. Siamese-network trackers provide strong learned appearance models, while correlation-filter trackers are computationally efficient and robust in moderate scenes. Each family fails in different situations.

This work proposes a learned weighted fusion framework that combines:

- Pretrained DaSiamRPN (Siamese tracker, unmodified)
- CSRDCF-style CSRT tracker (OpenCV reference implementation, unmodified)
- Per-frame fusion weights predicted by a supervised regression model
- Model selection among Gradient Boosting, Random Forest, and Extra Trees under leave-one-sequence-out cross-validation

The final box on each frame is `alpha * box_siam + (1 - alpha) * box_csrt`, where `alpha` is predicted for that frame from causal features only (no ground truth at inference time).

---

## Repository Structure

```
Hybrid-SiamRPN-CSRDCF-Tracker
│
├── README.md
├── requirements.txt
├── CITATION.cff
│
├── notebooks
│   ├── 01_Hybrid_SiamRPN_CSRDCF_Framework.ipynb
│   ├── 02_Baseline_Tracker_Comparison.ipynb
│   └── 03_Siamese_Tracker_Comparison.ipynb
│
├── figures
│   ├── baseline_success_curve.png
│   ├── baseline_success_ranking.png
│   ├── baseline_precision_curve.png
│   ├── baseline_precision_ranking.png
│   ├── fusion_success_curve.png
│   ├── fusion_precision_curve.png
│   ├── siamese_success_curve.png
│   ├── siamese_success_ranking.png
│   ├── siamese_precision_curve.png
│   └── siamese_precision_ranking.png
│
├── results
│   ├── baseline_tracker_comparison.png
│   ├── fusion_tracker_summary.png
│   ├── fusion_model_selection.png
│   ├── fusion_improvement_summary.png
│   ├── siamese_tracker_evaluation.png
│   └── siamese_fusion_improvement_summary.png
│
├── tables
│   ├── baseline_tracker_comparison.numbers
│   ├── baseline_tracker_per_sequence.numbers
│   ├── fusion_tracker_comparison.numbers
│   ├── fusion_tracker_per_sequence_results.numbers
│   ├── fusion_improvement_summary.numbers
│   ├── siamese_tracker_comparison.numbers
│   ├── siamese_tracker_sequence_results.numbers
│   └── siamese_fusion_improvement_summary.numbers
│
└── docs
    └── tracker_availability_summary.png
```

---

## Methodology

The framework has four stages, run on every frame:

1. Run the pretrained DaSiamRPN tracker.
2. Run the CSRDCF-style CSRT tracker.
3. Extract an 11-dimensional per-frame feature vector from the two trackers' outputs and recent history: the DaSiamRPN confidence score, the centre-distance disagreement between the two boxes, each tracker's deviation from a constant-velocity motion prediction, scale-ratio terms, an aspect-ratio disagreement term, and the frame's normalised position in the sequence.
4. Predict the fusion weight `alpha` with a fitted regression model and blend the two boxes.

**Training targets.** The correct weight for a frame is not directly observable, so training labels come from a ground-truth-guided grid search over 21 candidate values of `alpha` (0, 0.05, ..., 1.0) on training sequences only. Ground truth is used to build labels, never as an input feature.

**Model selection.** Gradient Boosting, Random Forest, and Extra Trees were compared under leave-one-sequence-out (LOSO) cross-validation. Gradient Boosting scored highest and was used for the reported fusion results.

**Auxiliary selector.** A separate "Late-Fusion Selector" variant (Safety-Selected Fusion in the paper) chooses among a fixed set of candidate boxes using a motion-consistency heuristic. It is reported separately from the raw learned fusion.

---

## Dataset

Experiments use a four-sequence pilot subset of the **OTB2013 benchmark**, each sequence truncated to its first 100 frames:

- Basketball
- Bolt
- Boy
- Car4

Ground-truth boxes follow OTB's per-frame `[x, y, w, h]` format. None of these sequences contains a sustained full-occlusion event.

Performance is evaluated using:

- Success AUC (area under the success curve over 101 IoU thresholds)
- Precision @ 20 pixels

---

## Experimental Results

All trackers were run directly under the same protocol (same sequences, same 100-frame cap, same metric code). Results are not taken from published figures.

### Aggregate results (four-sequence pilot subset)

| Tracker | Success AUC | Precision@20 | FPS |
|---------|------------:|-------------:|----:|
| **Learned Weighted Fusion (proposed)** | **0.794** | **1.000** | not measured |
| SiamRPN++ (ResNet-50) | 0.782 | 1.000 | not measured |
| Late-Fusion Selector (auxiliary) | 0.765 | 0.997 | not measured |
| Pretrained DaSiamRPN | 0.749 | 0.990 | not measured |
| CSRT | 0.738 | 0.995 | 32.8 |
| SiamFC (AlexNet) | 0.625 | 0.788 | not measured |
| KCF | 0.550 | 0.770 | 115.5 |
| MOSSE | 0.479 | 0.740 | 2.6 |

FPS was measured only for the three classical OpenCV trackers. The MOSSE figure reflects the implementation and setup used here, not the algorithm's expected speed.

### Per-sequence Success AUC

| Sequence | Learned Fusion | DaSiamRPN | CSRT | SiamRPN++ |
|----------|---------------:|----------:|-----:|----------:|
| Basketball | **0.677** | 0.598 | 0.480 | 0.626 |
| Bolt | **0.770** | 0.734 | 0.769 | 0.702 |
| Boy | 0.867 | 0.838 | 0.848 | **0.873** |
| Car4 | 0.863 | 0.826 | 0.855 | **0.928** |

### Fusion model comparison (LOSO average)

| Regressor | Success AUC | Precision@20 |
|-----------|------------:|-------------:|
| Gradient Boosting | 0.794 | 1.000 |
| Random Forest | 0.788 | 1.000 |
| Extra Trees | 0.787 | 1.000 |

### What these results show

- The learned fusion beats both of its own constituent trackers (DaSiamRPN, CSRT) on all four sequences, by 0.046 Success AUC over DaSiamRPN in aggregate.
- It narrowly exceeds a directly-run SiamRPN++ in aggregate (0.794 vs 0.782), but SiamRPN++ scores higher on two of the four sequences (Boy and Car4). The honest reading is that fusing two weaker trackers is competitive with a deeper single tracker, not that it is better in general.
- The auxiliary Late-Fusion Selector scores lower than the raw learned fusion (0.765 vs 0.794), reported here as a negative result.

---

## Baseline Tracker Comparison

Notebook `02_Baseline_Tracker_Comparison.ipynb` compares the conventional OpenCV correlation-filter trackers (CSRT, KCF, MOSSE) with the fusion on the same pilot set. Rankings and per-sequence results are in the `results/` and `tables/` directories.

---

## Siamese Tracker Comparison

Notebook `03_Siamese_Tracker_Comparison.ipynb` compares the fusion with standalone Siamese trackers on the same pilot set: SiamFC, SiamRPN++, and pretrained DaSiamRPN. The fusion scores 0.794 Success AUC against 0.782 for SiamRPN++; the margin is narrow and reverses on two of four sequences (see the per-sequence table above). SiamFC shows high variance across sequences (0.117 on Bolt to 0.893 on Boy), consistent with its lack of online adaptation.

---

## Limitations

1. **Small evaluation.** Four OTB2013 sequences, 100 frames each, not the full 51-sequence benchmark. Results are a pilot-scale trend pending validation on the full benchmark (and ideally OTB-100 or VOT).
2. **No full-occlusion events** in the chosen sequences, so re-detection after long occlusion is untested.
3. **Speed not profiled.** FPS was measured only for CSRT, KCF, and MOSSE. The real-time viability of the deep trackers and of the fusion pipeline is not established.
4. **No per-attribute breakdown** (occlusion, scale variation, illumination, clutter). Results are sequence-aggregated.
5. **Fragile margin over SiamRPN++.** The aggregate advantage is driven by two of four sequences (Basketball, Bolt) and is reversed on the other two (Boy, Car4) in the per-sequence comparison above.
6. **Model selection on the evaluation data.** Gradient Boosting was selected using the same four held-out sequences on which its results are reported, and its lead over Random Forest (0.794 vs 0.788) is small, so the headline figure is slightly optimistic.
7. **Small-sample cross-validation.** With four sequences, leave-one-sequence-out estimates have high variance.

---

## Running the Project

Clone the repository:

```bash
git clone https://github.com/hpk300494/Hybrid-SiamRPN-CSRDCF-Tracker.git
cd Hybrid-SiamRPN-CSRDCF-Tracker
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Execute notebooks in order:

1. `01_Hybrid_SiamRPN_CSRDCF_Framework.ipynb`
2. `02_Baseline_Tracker_Comparison.ipynb`
3. `03_Siamese_Tracker_Comparison.ipynb`

The notebooks expect the OTB2013 sequences and pretrained model weights at the paths set in their configuration cells; edit those paths for your environment.

---

## Results

The repository contains:

- figures of success and precision curves
- tables of aggregate and per-sequence results
- model-selection results
- per-sequence evaluations
- comparative tracker analysis

These results support the findings reported in the accompanying dissertation, with the scope described above.

---

## Citation

If you use this repository in your research, please cite:

```
Hanaa Parvez Khan.
Hybrid SiamRPN–CSRDCF Tracker for Robust Visual Object Tracking.
Master's Dissertation,
Liverpool John Moores University.
2024.
```

Additional citation information is available in `CITATION.cff`.

---

## Author

**Hanaa Parvez Khan**

M.Sc. Data Science
Liverpool John Moores University

---

## License

This repository is provided for academic and research purposes.
