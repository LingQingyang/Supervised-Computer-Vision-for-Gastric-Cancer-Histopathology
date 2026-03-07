# Gastric Cancer Histopathology Pipeline

A two-stage deep learning pipeline for **gastric cancer histopathology image analysis**, combining **lesion segmentation** and **image classification**.

This repository contains three Colab-exported Python scripts:
- **`gastric_cancer_u_net_train.py`**: train a U-Net model to segment lesion regions from RGB pathology images
- **`gastric_cancer_resnet_train.py`**: train a ResNet-based classifier on a 4-channel input composed of the RGB image plus a binary mask
- **`gastric_cancer_main.py`**: run end-to-end inference from image to predicted mask to final class label

---

## Project Overview

The core idea of this project is to use **segmentation as structural guidance for classification**.

Instead of classifying gastric cancer pathology images directly from RGB inputs alone, the workflow first predicts a lesion mask with U-Net and then concatenates the predicted mask with the original image. The resulting 4-channel input is passed to a modified ResNet classifier.

Pipeline:

1. **Input RGB histopathology image**
2. **U-Net** predicts a binary lesion mask
3. **RGB image + predicted mask** are concatenated into a 4-channel tensor
4. **ResNet18** predicts one of three classes:
   - `Normal`
   - `CancerType1`
   - `CancerType2`

This design aims to inject region-aware information into the classifier rather than relying only on global texture cues.

---

## Repository Structure

```text
Gastric_Cancer/
├── README.md
├── gastric_cancer_u_net_train.py       # U-Net training script
├── gastric_cancer_resnet_train.py      # ResNet training script
└── gastric_cancer_main.py              # End-to-end inference and visualisation
```

---

## Model Design

### 1. Segmentation Model

The segmentation stage uses **U-Net** from `segmentation-models-pytorch` with:
- **Encoder**: `resnet34`
- **Pretrained weights**: ImageNet
- **Input channels**: 3
- **Output channels**: 1
- **Loss**: `BCEWithLogitsLoss`
- **Optimizer**: Adam

Training performance is tracked with:
- training loss
- **Dice score**

### 2. Classification Model

The classification stage uses a modified **ResNet18** with:
- pretrained ImageNet initialization
- first convolution changed from **3 channels to 4 channels**
- final fully connected layer changed to **3 output classes**

The classifier takes:
- 3 RGB channels from the original image
- 1 binary mask channel from segmentation

Training performance is tracked with:
- training loss
- classification accuracy

---

## Data Format

The code assumes three components:

1. **Original pathology images**
2. **Binary masks** for lesion regions
3. **A CSV label file** containing:
   - `image_name`
   - `label`

Expected behaviour in the scripts:
- image file names and mask file names must match
- masks are loaded as grayscale images and binarized
- class labels are integer-encoded

---

## Preprocessing and Augmentation

The current implementation includes:
- resizing and square padding
- ImageNet normalization for RGB images
- nearest-neighbour resizing for masks
- synchronized image-mask augmentation

Augmentations include:
- random horizontal flip
- random vertical flip
- random rotation by `0°, 90°, 180°, 270°`

This keeps the geometric correspondence between image content and mask annotations.

---

## Training Workflow

### Step 1: Train the U-Net

Run:

```bash
python gastric_cancer_u_net_train.py
```

Outputs:
- U-Net training loss curve
- Dice score curve
- saved segmentation weights (`U_Net.pth`)

### Step 2: Train the ResNet classifier

Run:

```bash
python gastric_cancer_resnet_train.py
```

Outputs:
- classification loss curve
- accuracy curve
- saved classifier weights (`ResNet.pth`)

### Step 3: Run inference

Run:

```bash
python gastric_cancer_main.py
```

Outputs:
- predicted segmentation mask
- predicted class label
- class probabilities
- a 3-panel visualisation:
  - original image
  - predicted mask
  - classification result

---

## Environment

This project was originally developed in **Google Colab** and the scripts still contain Colab-specific components such as:
- `drive.mount('/content/drive')`
- Google Drive file paths
- notebook-export formatting
- shell commands such as `!pip install ...`

Recommended core dependencies include:
- Python 3.9+
- PyTorch
- torchvision
- segmentation-models-pytorch
- numpy
- pandas
- Pillow
- matplotlib
- tqdm

---

## Notes on Reproducibility

The repository currently contains code only.
It does **not** include:
- the training dataset
- trained model weights
- a packaged local configuration

To run the code locally, you will need to:
1. replace Google Drive paths with local paths
2. provide your own dataset in the expected format
3. save model weights to a local `./models/` directory or another custom path

Because the scripts were exported directly from Colab notebooks, a natural next step would be to refactor them into a cleaner project structure with:
- `requirements.txt`
- reusable dataset / model modules
- configurable paths
- train / inference entry points

---

## What This Repository Demonstrates

This project demonstrates practical experience with:
- **PyTorch-based medical image analysis**
- **U-Net segmentation**
- **CNN classification with structural priors**
- **histopathology image preprocessing**
- **custom dataset construction**
- **Google Colab prototyping for deep learning workflows**

More broadly, it reflects an attempt to combine **computer vision methods** with a **biomedical imaging task** in an interpretable, pipeline-based way.

---

## Future Improvements

Possible next steps include:
- adding a validation split and test evaluation
- reporting quantitative metrics more systematically
- packaging the code for local reproducibility
- saving example outputs directly in the repository
- replacing notebook-export scripts with modular Python files

---

## Disclaimer

This repository is a **course / project implementation repository**, not a clinical tool.
It is intended for learning, experimentation, and portfolio demonstration.

# 胃癌病理图像的分割与分类双阶段深度学习流程。

## 项目简介

本项目实现了一个面向胃癌病理图像分析的两阶段深度学习流程：先使用 U-Net 对病灶区域进行分割，再将原始 RGB 图像与预测得到的 mask 组合为 4 通道输入，送入 ResNet 进行三分类。

这个设计的核心思路是：先显式提取病灶区域，再把空间定位信息提供给分类器，从而让分类模型不只是“看整张图”，而是更聚焦于潜在病变区域。

## 方法概览

整体流程如下：

1. 输入胃癌病理图像
2. 使用 U-Net 生成病灶区域预测 mask
3. 将原始 RGB 图像与 mask 拼接为 4 通道输入
4. 使用 ResNet 输出分类结果
5. 可视化原图、预测 mask 与最终分类结果

## 仓库结构

```text
Gastric_Cancer/
├── gastric_cancer_u_net_train.py
├── gastric_cancer_resnet_train.py
├── gastric_cancer_main.py
└── README.md
```

### 文件说明

- **gastric_cancer_u_net_train.py**  
  训练 U-Net 分割模型，用于预测病理图像中的癌变区域。

- **gastric_cancer_resnet_train.py**  
  训练 ResNet 分类模型。分类器输入为 4 通道图像，即原始 RGB 图像加上 mask。

- **gastric_cancer_main.py**  
  端到端推理脚本。先运行 U-Net 生成 mask，再调用 ResNet 完成分类，并输出可视化结果。

## 模型设计

### 1. 分割模型

- 架构：U-Net
- 编码器：ResNet34
- 输入：3 通道 RGB 图像
- 输出：1 通道二值 mask
- 训练指标：Dice Score
- 典型用途：定位疑似癌变区域

### 2. 分类模型

- 架构：ResNet18
- 输入：4 通道图像（RGB + mask）
- 输出：3 个类别
- 训练指标：Accuracy
- 典型用途：根据原图与分割结果联合判断图像类别

## 项目特点

- **双阶段流程**：先分割、后分类，而不是直接端到端三分类
- **显式引入空间先验**：通过 mask 将病灶位置信息传递给分类器
- **适合教学与课程项目展示**：结构清晰，便于说明分割与分类如何协同工作
- **便于后续扩展**：可进一步替换骨干网络、加入更强的数据增强或尝试端到端联合训练

## 数据要求

本仓库当前主要展示模型实现与推理流程，不包含公开数据集文件。

运行本项目需要准备以下数据：

- 原始病理图像
- 对应的分割 mask
- 图像标签文件（用于分类训练）

并保证：

- 原图与 mask 文件一一对应
- 文件命名一致
- 标签文件能够正确映射图像名称与类别标签

## 运行说明

### 1. 训练分割模型

运行：

```bash
python gastric_cancer_u_net_train.py
```

### 2. 训练分类模型

运行：

```bash
python gastric_cancer_resnet_train.py
```

### 3. 运行端到端推理

运行：

```bash
python gastric_cancer_main.py
```

推理脚本会执行以下步骤：

- 加载训练好的 U-Net 与 ResNet 权重
- 对输入图像生成预测 mask
- 构造 4 通道输入
- 输出分类概率与预测类别
- 可视化原图、mask 和最终结果

## 依赖环境

本项目最初在 Google Colab 环境中开发，核心依赖包括：

- Python
- PyTorch
- torchvision
- segmentation-models-pytorch
- numpy
- pandas
- Pillow
- matplotlib
- tqdm

如需本地运行，建议先整理：

- 数据路径
- 模型权重保存路径
- Colab 专用代码（如 Google Drive 挂载）

## 当前仓库的边界

这个仓库更适合作为课程项目 / 医学图像学习项目的展示页，而不是一个已经完全产品化或可直接复现的研究代码仓库。

当前公开内容主要展示：

- 双阶段医学图像分析流程
- U-Net 分割与 ResNet 分类的组合思路
- 从训练到推理的基本实现框架

若要进一步提升可复现性，可以继续补充：

- 统一的数据目录结构
- `requirements.txt`
- 示例输入与输出图
- 训练结果指标汇总
- 更清晰的本地运行说明

## 适用场景

这个项目适合用于展示以下能力：

- 深度学习基础
- 医学图像处理
- 图像分割与图像分类
- PyTorch 模型训练与推理
- 将分割结果用于下游分类任务的 pipeline 设计

## 后续可扩展方向

- 将分割与分类做成联合训练框架
- 尝试更多分类骨干网络，例如 EfficientNet 或 ConvNeXt
- 引入更系统的数据增强策略
- 增加模型评估指标，如 precision、recall、F1-score、IoU
- 支持批量推理与结果导出
