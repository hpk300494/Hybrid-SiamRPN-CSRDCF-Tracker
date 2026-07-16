# Hybrid SiamRPN–CSRDCF Tracker

**Master's Dissertation Implementation**

This repository contains the implementation developed as part of my MSc Data Science dissertation at **Liverpool John Moores University**.

The project investigates whether combining a deep learning-based Siamese tracker with a correlation filter tracker can improve visual object tracking performance through a methodology-aligned hybrid fusion framework.

---

# Overview

Visual object tracking remains challenging under occlusion, appearance variation, scale changes, illumination changes, and fast motion.

Deep learning trackers such as **DaSiamRPN** provide robust semantic representations but can occasionally drift under difficult conditions.

Correlation filter trackers such as **CSRDCF** offer stable short-term localization and efficient online adaptation.

This dissertation proposes a **Learned Weighted Fusion** framework that combines the complementary strengths of both trackers.

Rather than replacing existing trackers, the objective is to investigate whether adaptive fusion can improve tracking robustness while maintaining high localization accuracy.

---

# Framework

The proposed pipeline consists of

```
Video Frames
      │
      ▼
+------------------+
|  DaSiamRPN       |
+------------------+
      │
      ▼
Bounding Box 1

Video Frames
      │
      ▼
+------------------+
|     CSRDCF       |
+------------------+
      │
      ▼
Bounding Box 2

             │
             ▼
+----------------------------+
| Learned Weighted Fusion    |
+----------------------------+
             │
             ▼
 Final Bounding Box
```

The fusion model combines the predictions from both trackers using learned confidence weights to generate the final object location.

---

# Repository Structure

```
Hybrid-SiamRPN-CSRDCF-Tracker/

│
├── notebooks/
│   ├── Fusion_Framework.ipynb
│   ├── Tracker_Comparison.ipynb
│   ├── Siamese_Comparison.ipynb
│   └── Evaluation.ipynb
│
├── figures/
│
├── tables/
│
├── results/
│
├── README.md
│
└── requirements.txt
```

---

# Evaluation Dataset

Experiments were conducted using representative sequences from the **OTB2013** benchmark.

Sequences evaluated include:

- Basketball
- Bolt
- Boy
- Car4

The experiments were performed using a controlled evaluation protocol with a maximum of 100 frames per sequence.

---

# Evaluation Metrics

Performance is evaluated using the standard OTB metrics:

- Success Plot (IoU)
- Success AUC
- Precision Plot
- Precision@20 pixels

---

# Experimental Results

## Comparison with Component Trackers

| Tracker | Success AUC | Precision@20 |
|----------|------------:|-------------:|
| Learned Weighted Fusion | **0.765** | **0.997** |
| Pretrained DaSiamRPN | 0.749 | 0.990 |
| CSRDCF | 0.738 | 0.995 |

The proposed fusion framework improved upon both individual component trackers under the same evaluation protocol.

---

## Comparison with Other Siamese Trackers

| Tracker | Success AUC | Precision@20 |
|----------|------------:|-------------:|
| SiamRPN++ | **0.782** | **1.000** |
| Learned Weighted Fusion | 0.765 | 0.997 |
| DaSiamRPN | 0.749 | 0.990 |
| SiamFC | 0.625 | 0.788 |

The proposed framework improves over DaSiamRPN and SiamFC but does not outperform the newer SiamRPN++ tracker.

---

# Key Findings

- Adaptive fusion successfully combines complementary tracking paradigms.
- The proposed framework consistently improves upon DaSiamRPN under the evaluated protocol.
- The framework also improves over CSRDCF.
- SiamRPN++ remains the strongest tracker among those evaluated.
- Results demonstrate that methodology-aligned tracker fusion is a viable approach for improving visual object tracking.

---

# Limitations

This repository represents the implementation used in a Master's dissertation and therefore has several limitations.

- Evaluation performed on four representative OTB2013 sequences.
- Maximum of 100 frames evaluated per sequence.
- Pretrained Siamese trackers were used.
- The work focuses on methodology validation rather than state-of-the-art benchmarking.

Future work includes evaluation on:

- Full OTB100
- LaSOT
- GOT-10k
- TrackingNet
- UAV123

and integration with modern transformer-based trackers.

---

# Requirements

Python 3.10+

Major libraries include

```
numpy
opencv-python
torch
torchvision
matplotlib
pandas
scipy
tqdm
```

---

# Citation

If you find this repository useful, please cite the associated Master's dissertation.

```
Hanaa Parvez Khan.

Methodology-Aligned Hybrid Visual Object Tracking Using Siamese Networks and Correlation Filters.

MSc Data Science

Liverpool John Moores University

2024
```

---

# Author

**Hanaa Parvez Khan**

MSc Data Science (Liverpool John Moores University)

Research Interests

- Visual Object Tracking
- Computer Vision
- Deep Learning
- Explainable AI
- Machine Learning Systems
- Edge AI

---

# License

This repository is provided for academic and research purposes.
