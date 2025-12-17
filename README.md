# OpenPCDet Error Model

本仓库用于从 OpenPCDet 在 KITTI 上的检测结果中：
1) 评估预测与 GT 的位置误差；
2) 为每个样本提取与点云/目标相关的特征；
3) 按特征统计误差分布与误差标准差（std）；
4) 基于特征（当前主要是 distance 与 intensity）学习/拟合 `std(distance, intensity)`，并进行可视化与对比。

## 目录结构

- [config.yaml](config.yaml)：推理/评估所需路径配置（例如 `cfg_root`、`model_path` 等）。
- Notebook
  - [errorEval.ipynb](errorEval.ipynb)：运行 OpenPCDet 推理并生成误差结果文件（如 `results.txt`）。
  - [featureExtract.ipynb](featureExtract.ipynb)：读取误差结果并结合 KITTI label/点云，提取特征（density、intensity、尺寸、confidence 等）。
  - [stdExtract.ipynb](stdExtract.ipynb)：对误差进行统计分析与分 bin 计算 std，并导出 `results/feature_std.txt`、`results/feature_error_std.txt` 等。
  - [errorDTModel.ipynb](errorDTModel.ipynb)：使用回归模型（RandomForestRegressor）拟合 std，并绘制热力图/散点图等。
  - [errorStatistcs.ipynb](errorStatistcs.ipynb)：误差整体统计（均值、方差、相关系数、(ex, ey) 计数热力图等）。
  - 其他：`errorBNNModel.ipynb`、`uncertainty_calibration.ipynb`、`errorStatistcs.ipynb` 等用于扩展实验/校准。
- [results/](results/)
  - `feature_error.txt`：误差与特征的原始导出（由特征提取流程生成/使用）。
  - `feature_std.txt`：按 (distance, intensity) 网格统计得到的 std 数据（`stdExtract.ipynb` 导出）。
  - `feature_error_std.txt`：每个样本关联其所在网格的 std（`stdExtract.ipynb` 导出）。
- [images/](images/)：各类分析图输出目录。
- [.vscode/settings.json](.vscode/settings.json)：本地 VS Code 配置。
- [Log.md](Log.md)：特征清单与简要记录。

## 数据流（推荐顺序）

1. **生成检测误差**
   - 运行 [errorEval.ipynb](errorEval.ipynb)
   - 产出：`results.txt`（或你配置的输出路径）

2. **提取特征**
   - 运行 [featureExtract.ipynb](featureExtract.ipynb)
   - 产出：包含误差 + 特征的文本（文件名以 notebook 内设置为准）

3. **计算误差 std（网格化）**
   - 运行 [stdExtract.ipynb](stdExtract.ipynb)
   - 产出：
     - [results/feature_std.txt](results/feature_std.txt)
     - [results/feature_error_std.txt](results/feature_error_std.txt)

4. **训练/拟合 std 模型并评估可视化**
   - 运行 [errorDTModel.ipynb](errorDTModel.ipynb)
   - 内容包括：
     - `RandomForestRegressor` 拟合 `std ~ f(distance, intensity)`
     - predicted/true std heatmap
     - std-std 对比散点图、std-error 散点图等

## 环境与依赖

Notebook 主要依赖：
- Python 3.8（见 notebook metadata）
- numpy / pandas / matplotlib
- scikit-learn
- OpenPCDet（用于推理/预测的 notebook：见 [errorEval.ipynb](errorEval.ipynb)）

## 结果文件说明（关键字段）

- `feature_error_std.txt` 表头示例（以实际导出为准）：
  - `error_x, error_y, px, py, density, intensity, std`

## 注意事项

- 路径通常在 notebook 中写死或从 [config.yaml](config.yaml) 读取，请根据本机数据集位置调整。
- `stdExtract.ipynb` 会对距离/强度做 binning，bin 数与最小样本数阈值会影响 `stdGrid` 的稀疏程度与平滑程度。