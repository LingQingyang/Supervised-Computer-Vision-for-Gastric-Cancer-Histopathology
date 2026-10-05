# Archived training results

[Project overview](../README.md) · [Download checkpoints](https://github.com/LingQingyang/Supervised-Computer-Vision-for-Gastric-Cancer-Histopathology/releases/tag/archived-training-results-v1) · [SHA-256 checksums](SHA256SUMS.txt)

These files were recovered from the project's original `模型结果.zip` archive. It contains **four PNG figures and two PyTorch checkpoints**. The files are published without altering their contents; training and inference were not rerun for this update.

## U-Net segmentation

Corresponding source: [gastric_cancer_u_net_train.py](../gastric_cancer_u_net_train.py).

![U-Net BCE training loss](figures/unet_training_loss_curve.png)

![U-Net training Dice](figures/unet_dice_score_curve.png)

The figures show decreasing segmentation loss and generally increasing Dice over 20 epochs. The final points are approximately **0.18 loss** and **0.83 Dice**, read visually from the plots.

In the supplied training script, BCE loss is averaged over minibatches. For Dice, logits are passed through sigmoid and thresholded at 0.5; the code calculates per-image Dice, averages within each minibatch, then averages the minibatch scores for the epoch. These are training-loop measurements, not a separate validation evaluation.

## ResNet18 classification

Corresponding source: [gastric_cancer_resnet_train.py](../gastric_cancer_resnet_train.py).

![ResNet18 cross-entropy training loss](figures/resnet_training_loss_curve.png)

![ResNet18 training accuracy](figures/resnet_accuracy_curve.png)

The figures show decreasing classification loss and generally increasing accuracy over 20 epochs. The final points are approximately **0.05 loss** and **98% accuracy**, read visually from the plots.

In the supplied training script, cross-entropy loss is averaged over minibatches. Accuracy is the number of correct argmax predictions divided by the number of training examples seen in the epoch. The classifier receives RGB images plus **annotated masks** during training; [inference](../gastric_cancer_main.py) instead uses **U-Net-predicted masks**. The displayed accuracy does not establish held-out performance for that two-stage inference workflow.

## Checkpoints and file inventory

The checkpoint names match the save/load filenames in the source scripts. They are available as original `.pth` files in the [archived training results Release](https://github.com/LingQingyang/Supervised-Computer-Vision-for-Gastric-Cancer-Histopathology/releases/tag/archived-training-results-v1).

| Original file | Size in bytes | Location |
| --- | ---: | --- |
| `U_Net.pth` | 97,904,027 | [Download segmentation checkpoint](https://github.com/LingQingyang/Supervised-Computer-Vision-for-Gastric-Cancer-Histopathology/releases/download/archived-training-results-v1/U_Net.pth) |
| `ResNet.pth` | 44,799,883 | [Download classification checkpoint](https://github.com/LingQingyang/Supervised-Computer-Vision-for-Gastric-Cancer-Histopathology/releases/download/archived-training-results-v1/ResNet.pth) |
| `unet_training_loss_curve.png` | 112,447 | [Original figure](figures/unet_training_loss_curve.png) |
| `unet_dice_score_curve.png` | 91,575 | [Original figure](figures/unet_dice_score_curve.png) |
| `resnet_training_loss_curve.png` | 105,623 | [Original figure](figures/resnet_training_loss_curve.png) |
| `resnet_accuracy_curve.png` | 104,678 | [Original figure](figures/resnet_accuracy_curve.png) |

Each figure is 1728 × 1361 pixels. [SHA256SUMS.txt](SHA256SUMS.txt) records all six original files: figure paths are relative to this directory, while checkpoint filenames refer to the downloaded Release assets. The same checksum file is attached to the Release.

## What the archive establishes

The archive records training progress for both models and preserves their saved weights. It contains **no raw epoch logs, run configuration, validation/test metrics, or saved inference visualisations**. The code defaults to five epochs, while the recovered plots span twenty; the exact settings and checkpoint selection rule for the plotted run cannot be established from these six files alone. No exact metric table has been reconstructed from the PNGs.

The original running guide's example class probabilities are illustrative and are not reported here as experimental results. The dataset and its class definitions are not included. No new generalisation or clinical-performance claim is made by this archival update.

## 中文说明

这里保存的是从原始结果压缩包恢复的训练结果。四张曲线与两个权重文件均保持原始内容，并提供 SHA-256 校验。图中的训练趋势可以直观查看，但压缩包没有原始日志、完整配置或独立测试结果；不能把训练准确率当作测试准确率，也不能据此确定实际临床表现。
