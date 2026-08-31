# 盖板参数与校准范围

> 关于盖板本身——即哪款盖板与哪种传感器配套、安装方式及其光学特性——已在**[DAQ用户手册](https://mapir.gitbook.io/daq)**中详细说明。 本页介绍如何向 Chloros *声明* 已安装的盖板，这正是确保校正准确的关键。

每个 DAQ 光传感器的出厂辐射校准所描述的都是*裸*传感器。安装在扩散板上的物理盖会改变传感器收集的光线，因此 Chloros 会在校准包的基础上应用出厂测得的 **盖校准曲线**。 声明正确的盖帽是获取校准数据的重要环节——本页面介绍了各型号可用的盖帽类型、如何进行声明，以及传感器实际的校准光谱范围。

## 各型号的盖板可用性

| 盖板配置文件 (`cap_id`) | 物理盖板 | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | 阳光余弦校正盖（**所有型号默认配备**） | 是 | 是 | 是 |
| `fov_15` / `fov_45` / `fov_90` | 视场限制锥（15° / 45° / 90°） | 是 | — | 是 |
| `fov_30` / `fov_60` | 视场限制锥（30° / 60°） | 是 | — | — |
| `none` | 未安装盖帽 | — | — | 是 |

型号特定说明：

* **DAQ-M 仅有一种盖帽型号：`sunshine_cosine`。** 其产品定义为“裸机加Sunshine顶盖”，而裸机版DAQ-M无需几何轮廓。
* **裸机版DAQ-U是真正的“裸机”** ——它完全不需要几何轮廓，因此不存在针对它的`none`轮廓。
* **DAQ-E 上的 `none` 并非“无操作”。** DAQ-E 的内嵌式、玻璃覆盖式漫射器本身具有实际的几何校正，因此“无顶盖”状态本身就是该型号的一个已测定轮廓。
* **未加装遮光罩的 DAQ-E 无法测量任何仰角的直射阳光** —— 阳光遮光罩是现场配置的必备组件。请勿计划使用未加装遮光罩的 DAQ-E 进行户外工作。

在图形用户界面（GUI）的单传感器设置中（“光传感器”选项卡中的齿轮图标），**盖帽**下拉菜单在 DAQ-U 和 DAQ-M 上也提供“无（裸传感器）”选项——根据上述说明，在这两种型号上，“裸”仅表示不应用盖帽校正。 仅在物理上拆下遮光罩时才应选择此选项。

## 声明遮光罩——及其重要性

**声明的 `cap_id` 必须与传感器上实际安装的遮光罩相匹配。** 无论是传感器还是软件，均无法检测到已安装的盖子。该声明决定以下两点：

1. 应用于每个光谱的 **实时校正**。
2. **写入每个 `.daq` 记录中的盖子标记**，后续反射率处理将以此为依据。

Sunshine 护罩按设计具有约 **12 倍** 的衰减效果，因此若声明的护罩型号错误，记录的光谱会按此倍数出现比例失真。请立即声明护罩变更。

### 设置护罩

GUI：光传感器选项卡 → 传感器行上的齿轮图标 → **护罩** 下拉菜单。 所有型号的默认值均为 `sunshine_cosine`（所有 DAQ 传感器出厂时均已安装余弦校正器），且该选择会随项目保留。

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI（后端必须正在运行）：

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI 在语法上支持完整的 `cap_id` 列表（例如 `{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`）； 每个配置文件在连接时都会根据传感器的型号进行验证，因此如果电容 ID 不可用（例如在 DAQ-U 上使用仅支持 E 模式的 ID），系统会明确报错而非进行错误校正。若未传入任何参数，后端的默认值为 `sunshine_cosine`。

Python SDK 注意： `cap_id` **并非** SDK 旋钮——`connect_daq_sensor()` / `DAQSensorSession` 不提供 cap 参数。 请通过上述 CLI 命令或 GUI 下拉菜单选择上限；请参阅 [SDK 参考](../reference/sdk-reference.md)。

高级：配置文件随 Chloros 安装包在 `daq/cap_profiles/<u|m|e>/<cap_id>.json` 中提供，用户可在 `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json` 中进行覆盖设置。

除了上限限制外，从未重新校准过的传感器会自动获得基于车队数据的小幅暗偏移优化——无需用户操作。

## 阳光上限性能（室外配置）

可供制定流程参考的数值：

| 属性 | 值 |
| --- | --- |
| 视场 | 180° 半球形 |
| 余弦响应误差 | 入射角达 60° 时 ≤ ±4%；达 70° 时 ≤ ±4.5% |
| 低日照限值 | 不建议在太阳高度角低于 ~15° 时使用 |
| 衰减 | ~12×（设计值） |
| 遮阳罩重新安装重复性 | ≈ 1.5 % |
| 定量辐照度 | **≥ 15 秒** 读数的平均值（仪器特性，非缺陷） |

对于任何定量辐照度数值（包括反射率参考值），请使用至少 15 秒读数的平均值，而非单帧数据。

## 已校准的光谱范围

| 属性 | 值 |
| --- | --- |
| 光谱采样 | 340–1010 nm，步长 5 nm（135 个点） |
| 辐射计量校准范围 | **~374–974 nm**（由软件强制执行） |

传感器报告完整的 340–1010 nm 网格数据，但可追溯至 NIST 的辐射增益范围为 ~374–974 nm。 对于任何在该范围内光谱权重不足一半的相机波段，Chloros **拒绝进行绝对反射率除法**，并报告跳过原因 `dls-uncalibrated-band-<nm>`，而非生成未经校准的产品。 在已上市的相机 SKU 中，仅 F988 滤光片超出此范围；它改用反射率面板工作流——请参阅 [反射率工作流](reflectance.md)。

有关传感器型号、传输方式和传感器 ID，请参阅 [DAQ 概述](README.md)。关于处理过程中 cap 标记的使用方式，请参阅 [记录与 .daq 格式](recording.md)。
