# Pt Nanoparticle Segmentation

贵金属纳米颗粒（Pt / noble-metal）在电镜图像上的**自动检测与分割**流程。基于 LoG 候选中心检测 + Meta 的 **SAM (Segment Anything Model)** 完成颗粒级分割，并输出统计结果。

## 功能简介

输入一张 STEM/TEM 的 DM4 图像，自动完成从预处理到颗粒分割、统计、可视化导出的完整 pipeline：

```
载入 DM4
  → 分离样品区/真空区 + 归一化
  → CLAHE 局部对比度增强
  → 基于区域的 LoG 候选中心检测（粗检 + 残差二次检测）
  → SAM 三种提示模式分割（point / rescue / box）
  → 三路结果合并（union）
  → 颗粒统计 + 结果导出
```

## 文件说明

| 文件 | 说明 |
|------|------|
| `Pt_nanoparticle_segmentation.ipynb` | 主流程 notebook，含完整代码与分步说明 |

## 依赖环境

Python 3，建议在带 GPU 的环境运行（SAM 推理较吃显存）。主要依赖：

```bash
pip install numpy scipy pandas matplotlib scikit-image hyperspy torch
pip install git+https://github.com/facebookresearch/segment-anything.git
```

| 库 | 用途 |
|----|------|
| `hyperspy` | 读取 `.dm4` 电镜数据 |
| `scikit-image` | CLAHE、LoG 检测、watershed、形态学处理 |
| `scipy` / `numpy` / `pandas` | 数值计算与统计 |
| `torch` + `segment-anything` | SAM 分割推理 |
| `matplotlib` | 可视化 |

> 运行前需下载 SAM 模型权重（如 `sam_vit_h_4b8939.pth`），notebook 的环境配置单元会处理下载/加载。

## 运行方式

1. 安装上述依赖。
2. 把待处理的 `.dm4` 图像路径填入 notebook 的载入单元。
3. 按顺序执行各单元（0 环境配置 → 1 预处理与候选中心 → 2 SAM 分割与导出）。
4. 显存紧张时，可在 2.1 单元用 `START / END` 分批跑，再用 2.1b 合并 chunk。

## 流程要点

- **预处理**：先区分样品区与真空区做归一化，再用 CLAHE 增强局部对比度。
- **候选中心**：按区域（厚/薄）分别做 LoG 检测，并加一轮残差检测找回漏检颗粒。
- **SAM 分割**：point（分批、可断点续跑）、rescue（找回漏检亮区 + watershed 拆分团聚）、box（按颗粒尺寸生成框）三种提示模式，结果取并集。
- **输出**：颗粒数量、尺寸等统计，以及填充/边界/实心三种可视化。
