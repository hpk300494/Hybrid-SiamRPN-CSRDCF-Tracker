# Hybrid-SiamRPN-CSRDCF-Tracker

A hybrid visual object tracking framework that combines a pretrained SiamRPN tracker with a CSRDCF-style correlation filter using learned adaptive fusion weights. The framework dynamically adjusts tracker contributions on a per-sequence basis through supervised machine learning to improve overall tracking robustness.

This repository accompanies the Master's dissertation:

**Hybrid SiamRPN–CSRDCF Tracker for Robust Visual Object Tracking**

---

## Overview

Visual object tracking remains challenging under occlusion, illumination variation, motion blur, scale variation, and background clutter. While Siamese-network trackers provide strong semantic representations, correlation-filter trackers remain computationally efficient and robust in certain scenarios.

This work proposes a learned weighted fusion framework that combines:

- Pretrained DaSiamRPN (Siamese tracker)
- CSRDCF-style CSRT tracker
- Machine-learning-based adaptive fusion
- Automatic model selection using Gradient Boosting

The framework predicts the optimal contribution of each tracker for every sequence and produces improved tracking performance over individual trackers.

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

The proposed framework consists of four stages:

1. Execute the pretrained DaSiamRPN tracker.
2. Execute the CSRDCF-style CSRT tracker.
3. Extract sequence-level tracking statistics.
4. Predict adaptive fusion weights using supervised machine learning.

Three regression models were evaluated:

- Gradient Boosting
- Random Forest
- Extra Trees

Gradient Boosting achieved the highest overall fusion performance and was selected for the final framework.

---

## Dataset

Experiments were conducted using sequences from the **OTB2013 benchmark**.

Example sequences include:

- Basketball
- Bolt
- Boy
- Car4

Performance is evaluated using:

- Success AUC
- Precision @20 pixels

---

## Experimental Results

### Proposed Hybrid Fusion

| Tracker | Success AUC | Precision@20 |
|---------|------------:|-------------:|
| Learned Weighted Fusion | **0.794** | **1.000** |
| Late-Fusion Selector | 0.765 | 0.997 |
| Pretrained DaSiamRPN | 0.749 | 0.990 |
| CSRDCF-style CSRT | 0.738 | 0.995 |

The learned weighted fusion framework achieved the highest Success AUC while maintaining perfect precision.

---

## Baseline Tracker Comparison

The repository also compares conventional OpenCV correlation-filter trackers:

- CSRT
- KCF
- MOSSE

Performance rankings and per-sequence results are provided in the `results/` and `tables/` directories.

---

## Siamese Tracker Comparison

The repository additionally evaluates multiple Siamese trackers:

- SiamFC
- SiamRPN++
- Pretrained DaSiamRPN
- Learned Weighted Fusion

These experiments demonstrate the effectiveness of adaptive fusion relative to standalone Siamese trackers.

---

## Running the Project

Clone the repository:

```bash
git clone https://github.com/<username>/Hybrid-SiamRPN-CSRDCF-Tracker.git
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

---

## Results

The repository contains:

- publication-quality figures
- formatted tables
- model-selection results
- per-sequence evaluations
- comparative tracker analysis

These results support the findings reported in the accompanying dissertation.

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
