# 🌿 Multispectral Semantic Segmentation for Precision Agriculture
## 📂 Dataset Access
* **Original Source:** [ETH Zurich WeedMap (RedEdge-M Multispectral)](https://www.research-collection.ethz.ch/entities/researchdata/0a5dcd76-cc44-4fbe-9f48-d8a717c90a30)
* **Cleaned & Ready-to-Use Dataset:** [Download my corrected version on Kaggle](https://www.kaggle.com/datasets/rishibihari/weedmap-rededge) 
  *(Note: This custom Kaggle version pre-applies the 16-bit label remapping and mask polarity fixes discussed above, making it instantly ready for PyTorch Dataloaders).*
  
**Live Interactive Demo:** [Click here to try the web app on Hugging Face](YOUR_HUGGING_FACE_LINK_HERE)

## 📌 Project Overview
In precision agriculture, automated weed eradication using UAVs or robotic ground vehicles requires highly accurate pixel-level separation of crops and weeds. While standard indices like NDVI easily separate green vegetation from bare soil, they fail to distinguish crop foliage from weed foliage. 

This major project implements a **1.94M-parameter Lightweight U-Net** using PyTorch to perform 3-class semantic segmentation (Background, Crop, Weed) on the **ETH Zurich WeedMap RedEdge-M Multispectral Dataset**.

### 🛠️ Key Contributions & Dataset Engineering
Training a model on the raw ETH Zurich dataset out-of-the-box yields a poor Weed F1 score of ~19.88%. During this project, I identified and resolved two critical, undocumented encoding bugs in the raw dataset archive:
1. **16-bit Ground-Truth Discrepancy:** The actual mask images encoded the "Crop" class as `10000` instead of the documented `1`.
2. **Inverted Mask Polarity:** The binary validity masks were inverted (`0` for valid field, `255` for invalid borders), which mistakenly wiped out 99.99% of valid crop pixels during standard loss calculations.

By engineering a custom PyTorch `Dataset` loader to remap the labels and correctly invert the ignore masks, the model's performance was drastically improved.

## 📊 Quantitative Results (Test Field 003)
The model was evaluated on over 13 million unseen valid pixels from Test Field `003`. 

* **Crop F1 Score:** 81.43% (97.49% Recall)
* **Weed F1 Score:** 68.31% (78.73% Recall)
* **Weed ROC-AUC:** 0.9827

### 🚫 Agronomic Risk Metrics
In real-world farming, accidentally spraying herbicide on crops is highly costly. The model is specifically optimized with inverse-frequency class weighting to minimize this risk:
* **Herbicide Damage Risk (Crop mistaken as Weed):** Only **0.80%**
* **Weed Escape Risk (Weed mistaken as Crop):** 6.63%

## 🔬 Spectral Ablation Study
A controlled ablation study was conducted to prove the necessity of multispectral data over standard RGB or 1-channel NDVI. The 6-channel full multispectral stack (`B, G, R, RedEdge, NIR, NDVI`) significantly outperformed the baselines:

| Configuration | Channels | Crop F1 | Weed F1 | Crop Damage Risk |
| :--- | :---: | :---: | :---: | :---: |
| Standard RGB | 3 | 53.86% | 55.64% | 0.82% |
| NDVI Index | 1 | 61.83% | 60.37% | 2.96% |
| **Full Multispectral** | **6** | **81.43%** | **68.31%** | **0.80%** |

*(See the `/RESULTS` folder for full visual prediction comparisons and confusion matrices).*

## 🚀 Future Scope & Edge Deployment
To bridge the gap between software simulation and real-world UAV deployment, the trained PyTorch weights (`.pth`) have been successfully exported into a hardware-agnostic computational graph (`.onnx` and `.onnx.data`). 

**Future deployment pipeline:**
1. Load the ONNX model onto an embedded companion computer (e.g., NVIDIA Jetson Nano / Raspberry Pi 4).
2. Interface the companion computer with a **Cube Orange+** flight controller running **ArduPilot**.
3. Trigger real-time precision spraying commands based on the U-Net's spatial segmentation map outputs during hexacopter flight operations.

## 📁 Repository Structure
* `/CODE` - Contains the full training Jupyter Notebook and Gradio web-app script.
* `/MODELS` - Contains the saved PyTorch `.pth` weights and optimized `.onnx` edge models.
* `/RESULTS` - Contains high-resolution prediction plots, confusion matrices, and ablation study charts.
