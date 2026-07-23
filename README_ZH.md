# Vision-and-Language Navigation in Continuous Environments (VLN-CE)

[项目主页](https://jacobkrantz.github.io/vlnce/) · [VLN-CE 挑战赛](https://eval.ai/web/challenges/challenge-page/719) · [RxR-Habitat 挑战赛](https://ai.google.com/research/rxr/habitat)

这是一个面向“视觉-语言导航”任务的开源代码库，目标是让智能体在连续空间中根据自然语言指令完成导航。项目支持两类数据集：Room-to-Room（R2R）和 Room-Across-Room（RxR），并提供基线模型与训练/评估脚本，方便研究者复现和扩展相关方法。

<p align="center">
  <img width="775" height="360" src="./data/res/VLN_comparison.gif" alt="VLN-CE 与传统 VLN 的对比">
</p>

## 项目简介

VLN-CE（Vision-and-Language Navigation in Continuous Environments）是一个在连续环境中进行指令引导导航的任务。与传统基于离散节点图的导航任务不同，VLN-CE 允许智能体在真实连续空间中自由移动，观测是视觉输入，动作更贴近真实机器人场景。

这个仓库提供了：

- 基于 Habitat 平台的环境和任务实现
- R2R 与 RxR 两类数据集支持
- 经典基线模型（如 Cross-Modal Attention 等）
- 训练、评估和推理脚本

## 主要特点

- 支持连续空间中的视觉语言导航任务
- 支持多种指令数据集，包括 R2R 和 RxR
- 提供可直接运行的基线代理与训练流程
- 适合做研究复现、模型训练与挑战赛提交

## 环境准备

本项目原始开发时基于 Python 3.6。建议使用 conda 创建虚拟环境：

```bash
conda create -n vlnce python=3.6
conda activate vlnce
```

### 安装 Habitat 相关依赖

VLN-CE 依赖 Habitat-Sim 0.1.7 和 Habitat-Lab 0.1.7。可以选择从源码安装，也可以通过 conda 安装：

```bash
conda install -c aihabitat -c conda-forge habitat-sim=0.1.7 headless
```

然后安装 Habitat-Lab：

```bash
git clone --branch v0.1.7 git@github.com:facebookresearch/habitat-lab.git
cd habitat-lab
python -m pip install -r requirements.txt
python -m pip install -r habitat_baselines/rl/requirements.txt
python -m pip install -r habitat_baselines/rl/ddppo/requirements.txt
python setup.py develop --all
```

### 安装本项目

```bash
git clone git@github.com:jacobkrantz/VLN-CE.git
cd VLN-CE
python -m pip install -r requirements.txt
```

## 数据准备

### 1. 场景数据：Matterport3D

项目使用 Matterport3D（MP3D）重建场景。需要先下载场景数据，并将其放置到以下路径：

```bash
# 需要使用 Python 2.7 运行
python download_mp.py --task habitat -o data/scene_datasets/mp3d/
```

数据应整理为：

```text
data/scene_datasets/mp3d/{scene}/{scene}.glb
```

通常需要 90 个场景文件。

### 2. R2R 数据集

R2R_VLNCE 是 Room-to-Room 数据集的连续环境版本。项目支持两个版本：

- R2R_VLNCE_v1-3
- R2R_VLNCE_v1-3_preprocessed

建议优先使用最新版本。可以参考仓库中的说明下载并解压到对应目录：

```text
data/datasets/R2R_VLNCE_v1-3
```

或：

```text
data/datasets/R2R_VLNCE_v1-3_preprocessed
```

### 3. RxR 数据集

RxR 数据集支持多语言指令（English、Hindi、Telugu），更适合研究多模态和跨语言导航。数据目录结构大致如下：

```text
data/datasets
└── RxR_VLNCE_v0
    ├── train
    ├── val_seen
    ├── val_unseen
    ├── test_challenge
    └── text_features
```

## 运行方式

主入口脚本是 run.py，支持训练、评估和推理：

```bash
python run.py \
  --exp-config path/to/experiment_config.yaml \
  --run-type {train | eval | inference}
```

### 示例：评估一个随机 agent

```bash
python run.py --exp-config vlnce_baselines/config/r2r_baselines/nonlearning.yaml --run-type eval
```

### 示例：生成 RxR 推理结果

```bash
python run.py \
  --exp-config vlnce_baselines/config/rxr_baselines/rxr_cma_en.yaml \
  --run-type inference
```

## 训练与评估

仓库中提供了两类训练器：

- DaggerTrainer：标准训练器，支持 teacher forcing 与 DAgger
- RecollectTrainer：基于数据集中的真实轨迹进行训练，不依赖 shortest-path expert

评估时可以通过设置 `EVAL.EPISODE_COUNT` 控制评估样本数；如果设置为 `-1`，则评估全部样本。

## GPU 配置建议

项目默认会优先使用可用的 GPU。对于训练和评估，通常建议：

```yaml
SIMULATOR_GPU_IDS: [0]
TORCH_GPU_ID: 0
NUM_ENVIRONMENTS: 1
```

这意味着模拟器和模型可以使用不同的 GPU，从而提高整体训练效率。

## 挑战赛说明

项目同时支持两个相关挑战赛：

- VLN-CE Challenge：基于 R2R 数据集的公开挑战
- RxR-Habitat Challenge：基于 RxR 数据集的多语言导航挑战

如果你希望提交结果，需要生成符合要求的轨迹文件，并使用仓库中提供的推理脚本输出结果。

## 许可证

本项目采用 MIT 许可证。训练模型和任务数据基于 Matterport3D 场景数据，相关使用需遵循 Matterport3D 的使用条款。

## 引用

如果你在研究中使用了本项目，请参考以下论文：

```tex
@inproceedings{krantz_vlnce_2020,
  title={Beyond the Nav-Graph: Vision and Language Navigation in Continuous Environments},
  author={Jacob Krantz and Erik Wijmans and Arjun Majundar and Dhruv Batra and Stefan Lee},
  booktitle={European Conference on Computer Vision (ECCV)},
  year={2020}
}
```

如果你使用 RxR 数据，请额外引用：

```tex
@inproceedings{ku2020room,
  title={Room-Across-Room: Multilingual Vision-and-Language Navigation with Dense Spatiotemporal Grounding},
  author={Ku, Alexander and Anderson, Peter and Patel, Roma and Ie, Eugene and Baldridge, Jason},
  booktitle={Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing (EMNLP)},
  pages={4392--4412},
  year={2020}
}
```
