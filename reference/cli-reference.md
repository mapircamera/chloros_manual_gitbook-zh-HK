# Chloros CLI 参考资料

**版本：**

1.2.0**生成时间：**2026-07-29 19:19 ·**修订时间：** 2026-08-30**受众：** 针对大型语言模型（LLM）使用进行优化；同时兼顾人类可读性。**范围：** `chloros-cli` 的所有面向用户的子命令，包含选项及可直接复制粘贴的示例。

本文档是随 MAPIR Chloros 发布的 `chloros-cli` 命令行工具的完整参考指南。本文档刻意做到详尽无遗，以便 LLM（或人类）无需查看源代码，即可根据下文列出的内容构建任何受支持的工作流。

如果您只需了解重点内容，请跳转至：
- [五分钟快速入门](#five-minute-quickstart)
- [LATTICE 相机首次连接工作流](#lattice-camera-first-connect-workflow)
- [DAQ 传感器首次连接工作流](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [采集模式、记录器与离线重新处理](#capture-modes-recorders--offline-reprocess)

---

## 命名约定

- 所有命令均以 `chloros-cli` 为前缀。在 Windows 上，二进制文件为 `chloros-cli.exe`；在 Linux /Jetson 上，二进制文件为 `chloros-cli`。
- 可选参数表示为 `--flag`。必需的位置参数不带方括号。
- 若提供了默认值，省略该标志时将使用该默认值。
- CLI 是一个基于 Chloros 后端（位于 `127.0.0.1:5000` 上的 Flask 服务器）的轻量级 HTTP 客户端。后端由大多数命令自动启动。`CHLOROS_BACKEND_URL=<url>`将**`lattice`**、**`project`**以及**`daq pool-*`** 命令族指向远程后端 ——核心命令（`process`、`login`、`logout`、`status`、`export-status`、`time-sync`、`selftest`）会刻意锁定 `http://127.0.0.1:<port>` 并忽略它 （IPv4 字面量可避免 Windows&#x27; `localhost`→`::1` 导致的每次请求约 2 秒的性能开销）。参见 [环境变量](#environment-variables)。
- 所有 SDK / CLI 调用均需使用 Chloros+ 账户登录（每台机器运行一次 `chloros-cli login`；结果缓存于 `~/.chloros/`）。
- 示例使用 Linux 路径；在 Windows 上，请将 `/home/user/...` 替换为 `C:/Users/.../...`。

---

## 顶级概要

```
chloros-cli [global options] COMMAND [command options]
```

### 全局选项

| 标志 | 说明 |
| --- | --- |
| `--backend-exe PATH` | 覆盖自动检测到的后端可执行文件。 |
| `--port N` | 后端HTTP端口（默认：`5000`）。 |
| `-v, --verbose` | 启用详细输出。 |
| `--restart` | 强制重启后端（终止所有正在运行的 `backend_server.py`）。 |
| `--version` | 显示版本信息（`Chloros CLI 1.2.0`）。 |
| `--help` | 显示顶级帮助。 |

### 命令索引

| 命令 | 用途 |
| --- | --- |
| [`process`](#chloros-cli-process) | 端到端处理Survey3或LATTICE捕获文件的文件夹。 |
| [`login`](#chloros-cli-login) | 使用 Chloros+ 账户对本机进行身份验证。 |
| [`logout`](#chloros-cli-logout) | 清除缓存的凭据。 |
| [`status`](#chloros-cli-status) | 显示当前许可证/身份验证状态。 |
| [`export-status`](#chloros-cli-export-status) | `process` 运行期间的 Live Thread-4 导出进度。 |
| [`language`](#chloros-cli-language) | 设置或列出 CLI 的显示语言（支持 38 种）。 |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | 默认项目文件夹（与 GUI 共享）。 |
| [`update`](#chloros-cli-update) | 检查并安装CLI更新（Linux /Jetson）。 |
| [`selftest`](#chloros-cli-selftest) | 系统诊断 + 基本功能测试。 |
| [`time-sync`](#chloros-cli-time-sync) | PTP 主时钟状态/控制。 |
| [`lattice`](#chloros-cli-lattice) | LATTICE 相机控制与采集（45 多个子命令）。 |
| [`daq`](#chloros-cli-daq) | DAQ 光谱传感器控制（DAQ-U / DAQ-M / DAQ-E）。 |
| [`project`](#chloros-cli-project) | 打开并运行已保存的Chloros项目（相机 + DAQ）。 |

---

## 安装

`chloros-cli` 随 Chloros 桌面安装程序在所有受支持的平台上提供——无需单独下载 CLI。安装平台包会将 `chloros-cli` 添加到您的 `PATH` 中，同时还会安装桌面应用程序及其 其驱动的后端二进制文件一同安装到您的 `PATH` 中。

最新下载： [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> 安装程序还附带了便捷的启动脚本（`Chloros_CLI.bat` / `Chloros_CLI.ps1`、`Launch_CLI.*`、`chloros-cli.sh`) ，这些脚本可打开一个可直接使用的CLI终端；相关内容已在[CLI用户指南](../CLI.md)中介绍，此处不再赘述。

### Windows (.exe)

1. 从下载页面下载Windows安装程序。
2. 运行`Chloros-Setup-x.y.z.exe`并按照向导操作。 默认安装路径为 `C:\Program Files\Chloros\`（CLI 位于 `C:\Program Files\Chloros\cli\`，安装程序会将其添加到 PATH 环境变量中）。
3. 打开一个新的终端（`cmd.exe`、PowerShell 或 Windows 终端），以便识别更新后的 `PATH`。

```powershell
chloros-cli --version
```

安装程序会自动将 `chloros-cli.exe` 添加到您的系统 `PATH` 中，并捆绑了 LATTICE 相机所需的 Arena SDK 运行时。

### Linux amd64 (.deb)

适用于 Ubuntu 22.04 LTS 或更高版本 / 基于 Debian 的 x86_64 工作站。

> **不支持 Ubuntu 20.04。**该软件包的依赖项列表源自
> 后端实际链接的库，其中包括 `libc6 (>= 2.34)`；
> Focal 系统自带 glibc 2.31。`apt` 会直接拒绝安装，而非在
> 运行时失败。

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

该 .deb 包安装了：
- 将 `chloros-cli` 升级至 `/usr/bin/chloros-cli`
- 将编译后的后端升级至 `/usr/lib/chloros/chloros-backend`
- Arena SDK 运行时（适用于 LATTICE 相机）
- 去噪模型、校准包、 以及更新通道配置

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

布局与 amd64 版 .deb 包相同，但包含针对 Jetson Orin / Orin NX / Orin Nano 优化的 CUDA 构建版本。

### 每台机器仅需认证一次

在调用 SDK / CLI 之前，每个平台都需要通过 Chloros+ 进行一次登录：

```bash
chloros-cli login user@example.com 'YourPassword'
```

凭据将缓存于 `~/.chloros/user_session.json` 中。

### 验证安装

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **需订阅 Chloros+。**CLI 需要有效的 Chloros+ 套餐。**Copper**是入门级 Chloros+ 套餐——所有付费的 Chloros+ 套餐均包含 CLI / SDK 访问权限；仅免费的**Iron** 套餐不包含此权限。（套餐ID映射：`0`=Iron/免费，`1`=Copper， `2`=青铜，`3`=白银，`4`=黄金。）升级请访问 [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing) 处进行升级。
>
> 此下限由后端强制执行，而不仅仅是CLI：SDK / CLI 标记的请求若未订阅付费套餐，无论其源自 `chloros-cli`、Python、SDK，还是自定义的 HTTP 客户端，均会因 `403 PLAN_UPGRADE_REQUIRED` 错误被拒绝。已注销的调用者则会收到 `401 AUTH_REQUIRED` 。在套餐的宽限期内（月套餐为30天，年套餐至到期日），离线访问仍可正常进行，宽限期结束后即停止； `chloros-cli status` 仍可正常工作，因此原因可见（这是 SDK / CLI 路径中唯一不受分级限制的例外——`GET /api/license-status`）。

---

## 五分钟快速入门

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

将一个图像文件夹通过完整的 Chloros 处理流程（目标检测 → 校准 → 晕影 → 反射率 → 索引导出）。

### 概述

```
chloros-cli process INPUT [OPTIONS]
```

### 位置参数

| 参数 | 描述 |
| --- | --- |
| `INPUT` | 包含 `.raw + .jpg`（Survey3）、`.tif/.tiff` (LATTICE) 或 `.dng` 文件的输入文件夹路径。 |

### 常用选项

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `-o, --output PATH` | 在默认项目路径下创建一个带有时间戳的新文件夹（除非已配置，否则默认为 `~/Chloros Projects`） | 要创建或重用的项目文件夹。如果该文件夹中已包含 `project.json`X，则会创建 `_1`/`_2` 作为并列文件夹，而非覆盖原有文件夹。 |
| `-n, --project-name NAME` | 自动（时间戳） | 项目名称。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` 使用Chloros+神经网络去拜耳算法；速度较慢但质量更高。 |
| `--vignette / --no-vignette` | `--vignette` | 暗角校正。 |
| `--reflectance / --no-reflectance` | `--reflectance` | 反射率校准（若检测到面板标靶则使用该标靶，对于 LATTICE 则采用 NIST 按序列号校准）。对于 LATTICE 多光谱数据，此功能同时兼作反射率 **产品**的切换开关——参见 [按产品导出切换开关](#per-product-export-toggles-lattice-multispectral)。 |
| `--ppk` | 关闭 | 应用来自 sidecar 文件的 PPK GNSS 校正。 |
| `--exposure-pin-1 MODEL` | 关闭 | 锁定Survey3双摄像头装置的“pin-1”模型。 |
| `--exposure-pin-2 MODEL` | 关闭 | 固定“pin-2”模型。 |
| `--recal-interval SECONDS` | 0 | 强制每 N 秒（以采集时间计）重新运行校准计算。 |
| `--timezone-offset HOURS` | 本地 | 覆盖输出元数据中内置的时区偏移量。 |
| `--format FORMAT` | `TIFF (16-bit)` | 选定以下选项之一：`TIFF (16-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)` 中的一个。 |
| `--indices NAME [NAME ...]` | 无 | 植被指数（`NDVI`、`NDRE`、`GNDVI`、`EVI`、`SAVI`、`OSAVI`、`CIG`、……）。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | 强制 LATTICE TIFF 文件的管道入口点（Survey3 中的 .raw 文件不受影响）。此外，还提供了一项“逃生通道”，允许对**不含原始数据**的捕获进行处理——请参阅 [捕获文件夹的结构示例](#what-a-captures-folder-looks-like)。 |
| `--debayered / --no-debayered` | 启用 | 输出线性去拜耳化结果（`Debayered_Images`）。参见 [按-产品导出开关](#per-product-export-toggles-lattice-multispectral)。 |
| `--preview / --no-preview` | 开启 | 输出显示预览 （`Preview_Images`）：RGB = 白平衡（如有可用，则采用DAQ光源；否则采用灰度世界） + 伽马；multispec = 伪彩色拉伸。 |
| `--radiance / --no-radiance` | 开启 | 输出 float32 辐射度 (`Radiance_Images`, W/m²/sr/nm)。 |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE反射率产品的参考基准：`auto` = 通过质量保证（QA）检查的帧内目标为绝对参考，DAQ向下辐射 (ρ = π·L/E) 备用方案；`target` = 严格模式（不采用 DAQ 替代）；`daq` = DAQ 权威模式。参见 [按产品导出切换选项](#per-product-export-toggles-lattice-multispectral)。|
| `--target-reflectance-dir DIR` | 无 | 各单元**测量**的目标反射率扫描的目录（`<serial>.csv`）；若未找到，则回退到标称的 T3/T4P 光谱。 |
| `--array-alignment / --no-array-alignment` | 开启 | LATTICE 阵列：将每个捕获数据中 `Chloros:Alignment*` XMP 文件中标记的模块间对齐信息应用于所有处理后的产品（去拜耳化 / 预览 / 辐射度 / 反射率 / 索引）。对于没有这些标签的图像，此操作不执行。 |
| `--array-alignment-crop / --no-array-alignment-crop` | 裁剪 | 将对齐后的导出数据裁剪至数组的公共重叠区域，以便所有模块共享同一覆盖范围；`--no-…` 保留完整的传感器画布（源外区域填充为黑色 源图像外））。 |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | 针对对齐变形进行重采样。`nearest` 保留精确的源数字密度（DN）值（不进行辐射值之间的像素混合）。 |

### 目标检测选项

| 标志 | 描述 |
| --- | --- |
| `--min-target-size PIXELS` | 探测器最小面板目标尺寸（像素）。 |
| `--target-clustering 0-100` | 聚类灵敏度。 |
| `--target / --targets` | 将输入文件夹视为仅含目标面板（跳过调查检测）。 |

### 示例

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### 按产品导出开关（LATTICE 多光谱）

LATTICE 处理会在 **单次处理中生成所有适用的产品**。四项按类型设置的开关——`--debayered`、`--preview`、`--radiance`、`--reflectance`——默认均**默认开启**；如需禁用其中一项，请使用 `--no-<type>` 格式。RGB 主摄像头仅输出去拜耳化后的图像及预览（不包含各波段的辐射度/反射率），因此 `--radiance`/`--reflectance` 对于它们而言是无操作的。对于 Survey3 `.raw`（遵循标准反射率/目标路径），这些开关将被忽略。*（旧的 `--radiometric-output {reflectance,radiance,sensor-response}` 标志已被 **移除** 并由这些开关取代； 已不再有 `sensor-response` 级别。）*

| 产品 | 输出 | 是否需要 DAQ 下行数据？ |
| --- | --- | --- |
| `--debayered` | 线性去马赛克 (`Debayered_Images`)。 | 否 |
| `--preview` | 显示预览 (`Preview_Images`)：RGB=白平衡 + 伽马校正；多光谱 = 假彩色拉伸。 | 否 |
| `--radiance` | 来自完整辐射测量链的 float32 单位 W/m²/sr/nm（`Radiance_Images`）。 | 否 |
| `--reflectance` | uint16 反射率 ρ（`32768` = 1.0），Pix4D 兼容。 | **是**，除非通过质量保证（QA）的帧内目标将其锚定 （见下文）。 |

`--reflectance-source` 选择反射率参考：**`auto`**（默认）将通过质量保证（QA）的帧内目标设为**绝对参考**——以目标为锚点的经验线链会在保留的面板上进行交叉评分，并应用测得的优胜结果——当无目标或质量检查失败时，则回退至数据采集（DAQ）向下分界值（ρ = π·L/E）；**`target`**采用严格模式（不进行数据采集替换）；**`daq`**选择采用 DAQ 权威行为。目标几何参数（ArUco / 固定 ROI / 条带）来自项目目标配置；`--target-reflectance-dir DIR` 通过目标单元的序列号/QR 保留每个单元的**实测** 扫描 （`<serial>.csv`），通过目标单元的序列号/QR进行查找，并以标称的 T3/T4P 光谱作为备用方案。

DAQ 反射率路径会自动从记录的 **`.daq`**（DAQ-U/M/E）**或与影像一同存储的 DAQ-M 原生 `.csv`**中自动解析出**时间戳匹配的向下辐射数据**。如果未在本地缓存相机级或 DAQ 校准包， 该处理流程将在首次使用时**从 AWS 自动获取**（仅需一次网络连接；缓存于 `~/.chloros/` 下）。

#### 读取反射率像素（Pix4D / Metashape / 您自己的脚本）

反射率以整数 DN 形式存储，且 **表示 ρ = 1.0 的 DN 值取决于源相机**：

| 源相机 | ρ = 1.0 对应值 | 如何判断 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（ρ 范围可达 2.0） | 文件中带有 XMP `Chloros:PixelScale=32768` 标记。 |
| Survey3 | `65535`（在 ρ 1.0 处被截断) | 无 `Chloros:*` XMP 标签——这种缺失*正是* 信号本身。 |

**读取 `Chloros:PixelScale` 并以此除**，而不是假设一个常数。该标签定义在 uint16 域中，因此即使在进行比例重放的输出格式中，它也保持不变——`32768` `TIFF (16-bit)`、`PNG (8-bit)`、 `JPG (8-bit)` 和 `TIFF (32-bit, Percent)` 均具有自描述性（需先将存储的数据类型归一化回 uint16：8 位数据乘以 257，浮点数乘以 65535）。

> **其中一种情况按设计不包含比例因子。**当将 8 位源捕获数据（BayerRG8）写入为 8 位TIFF 格式时，处理管道会将数据“裁剪”为 0..255 范围，而非进行重新缩放，因此所有高于 ρ≈0.008 的值都会被平滑处理为 255，且 该文件被描述为无缩放。Chloros 刻意省略了其中的 `Chloros:PixelScale` 和 `MicaSense:RadiometricCalibration` 元组，并记录了原因。**如果 LATTICE 反射率文件中缺少该标签，请勿假设存在比例尺——应以 16 位或 32 位重新导出**，而不是对原本不可分割的像素进行除法运算。

#### 导出时保留的 EXIF 信息

`process` 会将源捕获文件的 **GPS 块及其 ExifIFD**复制到每个产品中，因此
导出文件会包含 `FocalLength`、`FNumber`、`ExposureTime`、`ISO`、`DateTimeOriginal` 和
`CameraSerialNumber` 作为地理参考数据一同导出。

**`FocalLength` 对摄影测量而言是必不可少的。** Pix4D 通过
焦距加上飞行高度来计算地面采样距离；若缺少该标签，系统将退回到一个严重错误的比例尺。在一次
包含49张影像的橘园飞行任务中，由于缺少该标签，原本411米×160米的场地被重建为
47.8公里×13公里的区域 ——这张455 MP的正射影像中大部分区域显示为“无数据”，在有人检查GSD之前，这被误判为
拼接问题和BigTIFF问题。如果您的正射影像生成比例尺
不合常理， 请先对导出的产品运行 `exiftool -FocalLength` 命令。

此副本特意**未**使用 `-all:all`：IFD0 的结构标签在
复制时会破坏 LATTICE 的输出结果，且 `ExifImageWidth` / `ExifImageHeight` 被排除在外，因为它们描述的是
*源*捕获——否则，任何曾被调整过尺寸的导出文件都会携带与自身光栅
相矛盾的尺寸信息。XMP 是直接写入而非复制，因为 ExifTool
在复制XMP块时会丢弃同一调用中的XMP标签（否则将导致MAPIR
校准标签丢失）。

### 输出文件存放位置

生成的文件将写入**项目文件夹下，按相机分组，再按文件格式分组**：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE 相机的文件夹为 `LATT-<sensor>-<lens>-F<filter>`（与拍摄文件的 EXIF
`Model`），而Survey3的相机文件夹为`<model>_<filter>`——两台相机虽共用同一传感器和滤光片，但因
镜头不同而保持独立的文件结构，因为其暗角、视场和畸变各不相同。 文件夹格式
遵循 `--format`：`tiff16`、`tiff8`、`png8`、`jpg8`，或者 `tiff32` 对应
`TIFF (32-bit, Percent)`。

> **每个导出产品都保留源文件的名称。** 由
> `capture_…_raw.tif` 生成的 Radiance 导出文件仍名为 `capture_…_raw.tif` —— 它只是位于
> `tiff32/Radiance_Images/` 目录下。**该文件夹用于标识导出结果，而非文件名**，因此使用通配符
> 搜索 `*radiance*.tif` 不会找到任何结果；请改用目录进行匹配。

### 光传感器记录 — 已校准的 `.daq` + `.csv`

`process` 还处理 您的输入文件夹中处理 `.daq` 的记录，且此过程**无需**
任何图像数据：单独飞行的一台 DAQ-U / DAQ-M / DAQ-E 即可完成
完整的数据采集，而仅包含 `.daq` 文件的文件夹也是有效的输入。

DAQ 设备**无需**校准即可进行录制——这就是公开版
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) 记录器
(`record_daq.py`) 的默认行为：它们写入原始传感器计数并为文件添加时间戳，以便
Chloros **按序列号**（先从本地缓存，
再从MAPIR云端）获取该传感器的出厂校准值并加以应用。`process`将结果写回文件：

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv`每行记录一个读数：UTC时间戳、积分时间、总功率、
明视/暗视勒克斯、PPFD（及其蓝/绿/红分量）、峰值波长， 随后是
基于传感器自身波长网格的全光谱数据。`.daq` 会直接重新导入数据，无需
再次校准。

成功后，运行报告显示为 `Light-sensor products written: N (calibrated .daq + .csv)`。
括号中的内容描述了实际写入的数据，因此对于无文件包的传感器，显示为
`(RAW COUNTS — this sensor has no calibration bundle)`；对于包含 两者的文件夹。后端自身的
`[DAQ-EXPORT]` 和 `[RUN-SUMMARY]` 标题的措辞也是按此方式推导的——这
三者均不能将未经校准的原始导出数据称为“已校准”。

当 DAQ-U / DAQ-M / DAQ-E 记录，若其校准捆绑包无法获取——可能是
您处于离线状态，或是该传感器无存档校准数据——则会在
`[DAQ-EXPORT]` 行中**以具体原因跳过**，绝不会作为 包含原始计数值的“已校准”文件写入。
请连接互联网并重新运行。跳过原因即为读取器实际
针对该文件确定的原因（模式不可读、无校准包、写入错误），且运行
摘要会列出**独立的**原因——因单一原因被跳过的二十个文件会被记录为一个
，而非该原因的二十次重复。

#### DAQ-A 记录以原始计数形式导出

**DAQ-A** 系列早于按序列号分捆绑的系统，且没有可获取的校准捆绑包
——它是在现场通过反射率标靶进行校准的，这
正是它从未需要捆绑包的原因。拒绝这些记录导致用户无法获取其
数据，因此它们以**不同名称**导出：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

采用不同的文件名而非文件内部的标记，是因为该声明必须能够
在仅以文件名形式通过电子邮件转发时保持有效。`.csv` 文件头声明
`raw spectral sensor counts (NOT irradiance)`，并警告说这些值仅在
**文件内部**可比——这正是基于目标的校准使用它们的目的——而
不能跨传感器比较。 与功率相关的光度学列（总功率、明视和
暗视勒克斯、PPFD）被写为 **NULL**，而非通过计数积分得出，且运行
摘要显示为 `RAW COUNTS`，因此 “导出”到日志中的数据不能被解读为辐照度。

旧版 **v1.01 / v1.02** 记录（由 DAQ-A-SD 写入）不包含每次读数的时戳，
仅包含文件的写入时间。图像↔下行光匹配器仍然拒绝这些数据——将
帧与写入时间进行匹配会导致隐性错误——但导出工具可以读取它们，且
CSV会输出`clock=daq_created_on`，因此该产品会标明其使用的时钟类型。

### 注意事项

- `process` 会自动检测您的文件夹是 Survey3、LATTICE 还是混合类型。
- 进度流通过 Server-Sent Events；CLI 显示各线程的实时进度（检测中、分析中、处理中、导出中）。
- 对于 Linux /Jetson，CLI 会检查交换空间，并在处理大型文件夹前可能发出警告。支持纹理的去拜耳算法还会在低功耗 Jetson 设备（Nano、Orin Nano）上自动应用 GPU 频率限制。
- 运行 运行成功时，报告会显示写入的图像产品数量（`Image products written: N`）。

#### 未写入任何图像的运行将视为失败

如果您请求了图像产品，但运行结果为**零**——仅输出 `project.json` 和
`calibration_data.json` ——则 `process` 会将此视为失败：它会输出
`Processing finished but wrote no image products.` 并**以非零状态退出**，因此脚本可以
检测到此情况。该消息会列出项目文件夹及常见原因：

- 输入文件夹未被识别为捕获 （请检查布局并参考 `--input-level`），或者
- 所有请求的产品均因不适用于这些相机而被跳过（例如，请求
  从仅支持RGB的相机获取辐射度/反射率）。

请使用`--verbose`重新运行，并检查后端日志中的`[LATTICE-EXPORT]` / `[EXPORT-CHECK]`行，
这些行会针对每台每台相机的跳过情况，这些情况通常不会出现在CLI的输出中。

一次有意的仅元数据运行——所有产品选项均关闭且没有`--indices`——仍然是
**成功的**，因为在此情况下输出空图像才是正确结果。**仅光传感器运行**也是如此：根据定义，一个包含 `.daq` 记录文件的文件夹本就没有图像可导出，
因此该运行将根据其生成的校准 `.daq` / `.csv`文件来评判。

---

## `chloros-cli login`

使用 Chloros+ 云账户对这台机器进行身份验证。凭据已安全缓存于 `~/.chloros/user_session.json` 中。

```
chloros-cli login EMAIL PASSWORD
```

### 示例

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` （从密码中截取或复制部分内容）。遇到 401 错误时，CLI 会自动重试：先在原密码后追加 `$$`，再尝试使用去重后的密码一半；若重试成功，则登录并输出正确的单个引号语法供下次使用。

> **无界面/脚本使用：无缓存会话意味着会出现交互式提示符，而非快速失败。** 任何会启动后端的命令（`process`、`status`、`export-status`、`time-sync`、…） 若在未缓存许可证/会话的情况下运行，会在继续执行前通过标准输入（stdin）进入交互式 `Email:` / `Password:` 提示符。因此，没有缓存会话的无人值守作业会因等待输入而挂起——在安排无头作业之前，请在每台机器上运行一次 `chloros-cli login EMAIL PASSWORD` 一次。

---

## `chloros-cli logout`

清除缓存的会话，并强制在下一次调用时重新登录。

```bash
chloros-cli logout
```

---

## `chloros-cli status`

显示当前许可证等级（铁/铜/青铜/银/金）、经过身份验证的用户以及设备绑定次数。

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

轮询 Thread-4 实时导出进度。在另一个 shell 中运行 `process` **期间** 调用此命令是安全的。

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

设置CLI的显示语言（支持38种语言， 包括中日韩、从右到左及印度语系）。在无法渲染该脚本的旧版控制台上，将优雅地回退为英语。

```
chloros-cli language [LANG_CODE] [--list]
```

### 示例

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## 项目文件夹命令

这些命令用于管理默认项目文件夹的位置（与 GUI 共享）。

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ 仅限 Jetson。检查 `version_url` 是否来自 `/etc/chloros/update.conf`，并提供下载和安装匹配的 `.deb`。

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

在 Linux /Jetson 上，CLI 还会在**每次启动时自动检查更新**（非阻塞式，绝不会延迟命令执行）：它读取 `/etc/chloros/update.conf`，将结果缓存 1 小时在 `~/.chloros/update_cache.json` 中，并在存在新版本时输出 `Update available: vX.Y.Z / Run: chloros-cli update`。遇到任何错误或在 Windows 时，将静默跳过。

---

## `chloros-cli selftest`

运行一个包含 7 个步骤的初步测试：版本检查、端口可用性、后端启动、`/api/test`、 `/api/system-info`（GPU/CUDA/PyTorch）、去噪模型是否存在、CUDA+去噪器就绪状态。

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP主节点的状态与控制。Chloros主机运行PTP主节点；LATTICE摄像机和DAQ-E单元作为从属节点，用于生成跨设备时间戳。

| 子命令 | 描述 |
| --- | --- |
| `status` | 显示主时钟状态、BMCA优先级及时钟标识。 |
| `peers` | 列出通过 Delay_Req 检测到的从设备（摄像头 + DAQ-E 传感器）。 |
| `cameras` | 每台摄像机的 PTP 健康状态（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`）。 |
| `restart` | 重启主控进程。 |
| `set-priority --priority1 N --priority2 N` | 覆盖 BMCA 优先级。 |

### 示例

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE 摄像头控制。每个子命令都通过Chloros后端进行路由；该后端拥有摄像头池，因此后续的CLI调用将复用同一个打开句柄。

### 常用选项（大多数子命令共用）

| 标志 | 说明 |
| --- | --- |
| `-d, --device N` | 摄像头索引（默认：0）。 |
| `-d, --device N` | 摄像头索引（默认：0）。 |
| `-s, --serial SN` | 特定序列号；覆盖 `--device`。 |
| `--serials SN1,SN2,…` | 用于多摄像头操作的以逗号分隔的序列号。 |
| `--all` | 对所有已发现的摄像头进行操作。 |
| `--exposure US` | 曝光时间（单位：微秒）。 |
| `--gain DB` | 以分贝（dB）为单位的增益。 |
| `--pixel-format FMT` | 例如：`BayerRG8`、`BayerRG12`。 |
| `--width N` / `--height N` | 图像尺寸。 |
| `--preset {default,high_quality,high_speed,triggered}` | 应用预设。 除 `triggered` 外，其余均为自由运行模式；该模式会在第 2 条线上检测到硬件边沿时触发摄像机——如果该线没有信号驱动，摄像机将无限期等待而非进行拍摄。 |
| `-o, --output DIR` | 输出目录（默认：`output`）。 |
| `--packet-size {auto,jumbo,standard,N}` | GVSP 数据包大小。 `auto` 运行 ICMP+GVSP 探测；`jumbo` = 9000；`standard` = 1500。 |

### LATTICE 摄像头首次连接工作流

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### 子命令参考

#### 发现与信息

| 子命令 | 用途 |
| --- | --- |
| `lattice info` | 列出已连接的摄像头（厂商、型号、序列号、IP、MAC）。 |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | 分析主机系统以获取最佳摄像头配置。 `--no-discover` skips 摄像头发现（更快，仅分析网卡）。 |
| `lattice network [--fix] [--estimate] [--cameras N]` | 检查/修复网卡设置；估算带宽/帧率。 |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | 稳定架构后端网络能力 + 阵列推荐 （返回 `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`)。 `auto_capped_fps` 保持请求的分辨率，但限制目标帧率——读取 `recommended.recommended_target_fps` 并将其作为连接目标传递；将其视为成功，而非错误。 |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | 无需打开摄像头即可进行假设分析。**`--n-active` 表示网络上的摄像头总数，而不仅限于此数组中的**——当独立摄像头并发流式传输时，或网络带宽预算的计算基于未（默认值：`len(--models)`）。始终打印聚合值 `Wire budget:`（MB/s 需求值与防冲突上限）和 `Max cameras:` 行，并在数组对总线超额订阅时标记 `**OVER-SUBSCRIBED**` 标记为红色——参见 [数组 fps 与突发模型](#array-fps--burst-model)。 |
| `lattice gpu` | 显示 GPU 状态。 |
| `lattice firmware [--update] [--force] [-y\|--yes]` | 检查或更新摄像头固件。本地 `.fwa` 选项被锁定：当存在与该构建版本的 `firmware/<MODEL_PREFIX>/` 中的文件若与该构建的 `MIN_FIRMWARE_VERSION` 匹配，则会在存在时被刷入（仅将最高版本作为备用方案），因此存储在磁盘上的较新供应商镜像在该引脚被触发前处于非活动状态——较新版本会通过签名的 AWS 清单有意地推送至设备，当有较新. |
| `lattice presets [--apply NAME]` | 列出或应用摄像头预设。 |
| `lattice status` | 摄像头实时状态。 |

#### 捕获

| 子命令 | 用途 |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | 单帧。**默认保存所有导出类型**（`--processing all`）；参见 [捕获导出级别](#capture-export-levels-the-all-default)。 `--levels` 保存一个明确的子集 （覆盖 `--processing`）；`--force-daq` 会将分配的 DAQ 读数写入 `.daq` 旁路文件，即使在仅原始数据的抓取中也是如此。`--jpeg-quality` = JPEG 质量 1–100 （默认 95）。 |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | 持续将数据流写入磁盘直至按下 Ctrl+C。 |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | 基于浏览器的实时 MJPEG 预览。 `--ae-damping` 设置自动曝光阻尼（0.4–100）。 |

#### 传感器调校

| 子命令 | 用途 |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | 读写任何 GenICam 节点。 |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | 曝光与自动曝光。 |
| `lattice gain [--auto] [--off] [--set DB]` | 增益与自动增益。 |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | 传感器ROI与像素合并。 |
| `lattice format [--set FMT] [--list]` | 像素格式。 |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | 硬件/软件触发。 |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]`（无标志 = 单次白平衡） | 白平衡操作。RGB仅限拜耳阵列相机；在单色M3M上为无操作（跳过）。|
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB显示色彩处理管道。`natural` （默认）是低成本的实时处理；`enhanced` 添加了去色边 + 鲜艳度 + CLAHE 局部对比度，以实现完整的 Hub-Parity 效果，每帧处理成本约为 2 倍，因此会降低 **实时** 帧率——无论哪种方式，保存的截图始终会获得完整的处理效果。RGB /仅限拜耳传感器相机；在单色 M3M 上跳过。 |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | 显示饱和度/对比度（RGB滤镜相机）。在单色 M3M 上跳过。 |
| `lattice filter [--set NAME] [--list]` | 设置相机的滤镜模型（`RGN-IMX265`、`OCN`、`NGB`、……）。 |
| `lattice power [--sleep]` | 检测电源/热节点；切换低功耗待机模式。 |

#### 校准与传感器

| 子命令 | 用途 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 通过反射率标靶进行校准。 |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | 内置下射光传感器命令。 |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 对现有图像应用暗角校正。 |

#### 多摄像头 （瞬态会话）

| 子命令 | 用途 |
| --- | --- |
| `lattice multi-info` | 列出所有具有同步角色的相机。 |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | 从每台相机获取一帧同步帧。当连接持久性阵列时，默认保存 **默认导出所有类型**；当连接持久化数组时，临时无数组的备用方案仅支持**去拜耳处理**（如需处理其余内容，请先运行 `array-connect`）。 |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | 流式传输同步帧（临时）。 |
| `lattice multi-test [--count N]` | GPIO 同步时序测试。 |
| `lattice multi-detect [--line LINE] [--json]` | 自动检测 GPIO 主从接线。 |

#### 对齐

| 子命令 | 用途 |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — 加上检测器/匹配器调节旋钮 `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`、RANSAC 调节旋钮 `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`、多帧组合 `[--averaging mean\|median\|inlier_weighted]`、几何约束 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`、空间限制 `[--roi X0,Y0,X1,Y1] [--mask PATH]` 以及按从属节点覆盖设置 `[--per-cam-override SN:KEY=VALUE]`（可重复） | 根据实时摄像头数据计算对齐配置文件。 `--prefilter` 默认采用 `gradient`（边缘图；与 GUI/阵列对齐器一致——边缘在各光谱波段间得以保留）。 `--matcher flann` 在特征数超过 ~5000 时效果显著；`--averaging median` 对单次异常采集具有鲁棒性，`inlier_weighted` 按匹配次数进行加权； `--lock-scale` 投影至最近的旋转角度（不调整比例）；`--lock-axis` 将一个平移分量设为零；`--mask` 适用于所有相机（使用 `--per-cam-override` 进行单相机设置，例如`--per-cam-override 214701292:method=phase`）。`--rms-threshold-px` 拒绝保存重投影均方根误差超过阈值的校准。|
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | 捕获一帧对齐的多波段图像。`--bit-depth` 默认匹配相机；`--no-crop` 保留全帧 （用黑色填充）；`--interpolation`（默认 `linear`）和 `--border-mode`/`--border-value` （默认 `constant`/0）控制 CPU 变换——无论如何，GPU 路径均为双线性变换。 |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | 流对齐多波段帧（与 `align-apply` 具有相同的 Warp 控制项）. |
| `lattice align-info --profile PATH [--json]` | 显示配置文件详细信息。 |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | 更改图层顺序。 |

#### 索引 / 植被数学

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

完整标志集：`--input PATH | --live --profile PATH`、`--preset NAME`（NDVI / NDRE / EVI / SAVI / GNDVI /…）、`--formula EXPR`、`--channel SYM=BAND` （可重复），`--capture-level raw|debayered|radiance|reflectance|unknown`（覆盖源文件TIFF中记录的捕获级别；默认：从TIFF元数据中读取），`--output PATH`，`--output-format all|raw|tif|colorized|lut|png`， `--gradient NAME|JSON`、`--vmin/--vmax/--percentile LO,HI`、`--bg-mode clip|transparent|indexColor|backgroundColor`、`--colorize`、`--list-presets`、 `--list-gradients`。 对于 `--live`，对齐变形旋钮同样适用：`--save-multiband`、`--gpu/--no-gpu`、`--no-crop`、`--bit-depth 8|12|16`、`--vignette`、`--interpolation nearest|linear|cubic|lanczos`、`--border-mode constant|replicate|reflect|wrap`、`--border-value N`。

> **`--channel` 符号区分大小写。** 符号部分必须与预设的通道名称完全一致（预设使用小写，例如：NDVI = `red`,`nir` — 请检查 `--list-presets`）， 且频段部分必须与对齐堆栈中的频段名称匹配（或在离线模式下为基于 0 的频段索引）。`--channel red=Red_660 --channel nir=NIR_850` 有效；`--channel RED=660` 将因 `channel_map missing entries` 错误而失败。

#### 持久连接（Smart-Prep，等同于 GUI 的流程）

这些命令可在多次调用 `CLI` 时，使后端池中的摄像头保持打开状态。

| 子命令 | 用途 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | 将一台摄像机添加到池中（单摄像机，非数组）。 |
| `lattice cam-disconnect [--serial SN] [--all]` | 释放。 |
| `lattice cam-list` | 列出池中的相机。 |
| **`lattice array-connect`**|**连接一个持久化同步阵列（推荐的入口点）。** 运行完整的 GUI 智能预处理流程。 |
| `lattice array-disconnect [--array-id ID] [--all]` | 释放一个阵列。 |
| `lattice array-list` | 列出已连接的阵列。 |
| `lattice array-status [--array-id ID]` | 实时帧率、PTP、最后一次错误。 |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | 从实时阵列中进行一次同步捕获 — 单次 / 连续 / 间隔 / 最快。 **默认值为 `all`**（每台相机每种适用的导出类型生成一个文件）。被跳过的相机（例如RGB不包含在辐射度/反射率）的摄像机将通过 `Skipped: SN:<serial> (<reason>)` 报告；用于反射率测量的 DAQ 读数将与之一同保存，并通过 `DAQ: <path>` 报告。 参见 [采集模式、记录器与离线重处理](#capture-modes-recorders--offline-reprocess)。 |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | 将实时组合索引视图录制为视频/GIF（监控级；需打开组合流）。 |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | 高帧率原始拜耳连拍（分析级；需离线重新处理）。 |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 将已保存的原始连拍数据重新处理为校准后的视频。 |

##### `array-connect` 选项

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 自动发现所有 LATTICE 摄像头（需 ≥2 个） | 首个串行号为 MASTER。若省略，则仅发现 LATTICE（`TRI032*`）型号并连接所有设备。 |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 同步线。 |
| `--target-fps F` | 自动 | 主控器触发频率。 |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | 覆盖层级选择器。 |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **主机的的持续带宽预算（单位：MB/s）——整个数组分配均以此数值为基准。**当数组报告GVSP损坏帧时，请降低此值：自动值源自网卡报告的链路速率，该数值往往高估了USB适配器、带宽有限的PCIe通道以及繁忙的共享互连结构的实际带宽。该值会保存在项目的数组捕获 块中，因此通过重新打开 / CLI / SDK 重新连接即可恢复该值。参见 [阵列健康状况](#array-health--which-subsystem-is-losing-frames)。 |
| `--binning {1,2,4}` | 自动 | 硬件分组。 |
| `--no-recommend` | 关闭 | 跳过网络分析步骤。 |
| `--no-ptp` | 关闭 | 禁用 PTP（此时跨摄像机时间戳将**无法**进行比较）。 |

### 智能自动曝光（Smart-AE）/ 智能捕获（Smart-Capture）

LATTICE 阵列一经连接便会在后台持续运行自动曝光（AE），但新定位的场景需要片刻时间才能收敛。`array-capture --smart` 是一个 **便捷的集成解决方案**：它会等待阵列中所有摄像头的自动曝光稳定后，再触发拍摄。在会话中途切换场景时请使用此选项。

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

默认的稳定策略较为保守：5 秒超时，1.5 秒稳定性窗口，±5% 曝光分布容差。若需与自动化设置不同的行为，请通过SDK（`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`）进行调整。

### 拍摄导出级别（`all` 默认设置）

自本版本起，`lattice capture`、`lattice multi-capture` 和 `lattice array-capture` **默认设置为 `--processing all`** — 每种导出类型对应一个保存文件，适用于每台相机，与 GUI 中的“捕获全部”行为一致。级别如下：

| 级别 | 输出 | 适用对象 |
| --- | --- | --- |
| `raw` | 直接来自传感器的单通道拜耳（单色摄像头：单波段）数据。 | 所有摄像头。 |
| `debayered` | 3 通道 BGR 去马赛克（单色摄像头： 1通道灰度）。 | 所有相机。 |
| `radiance` | 通过完整辐射测量链获得的float32 W/m²/sr/nm。 | 仅限多光谱（M3C/M3M）——**RGB滤光片相机将跳过此模式**。 |
| `reflectance` | uint16 ρ（`32768` = 1.0），支持 Pix4D。 | 仅限多光谱， 且**仅当已绑定数据采集设备（DAQ）且相机已校准时**有效；否则跳过。 |
| `preview` / `display` | 完整的GUI预览处理链（CCM + 白平衡 + 根据相机配置文件设置的伽马校正）。`lattice capture`将此命名为`preview`；`array-capture`/`multi-capture` 使用 `display`。 | 所有摄像头。 |

传递单个级别参数可仅保存该级别（`--processing debayered`）。当请求 `all` 时，不适用于特定凸轮的级别将被跳过（并报告），不会报错——未连接或未校准的摄像头仍会获得 `raw` / `debayered` / `preview`。

对于任何反射率帧，实际使用的 DAQ 下行读数会被写入图像旁边的 **`.daq`** 旁载文件中（以便日后可重新处理该捕获数据），并在 `DAQ:` 行中报告。

### 捕获文件夹的结构示例

每种导出类型都会存放在其**独立子文件夹**，因此多级捕获中绝不会混淆不同类型：

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` 表示捕获时间戳，而 `<serial>` 表示相机序列号，因此一个同步组在所有相机间共享
同一时间戳。 **请注意一个不对称之处：** `display` 层级的文件存储在
名为 `preview/` 的文件夹中，而文件名本身保留 `_display` —— 文件夹名和后缀仅
。未知级别将回退到以其自身名称命名的文件夹中，如果子文件夹
无法创建，文件将写入输出根目录，而非丢失。

**关于-处理“captures”文件夹：**将 `chloros-cli process` 指向**“captures”根目录**
（`output/`）。`process` 通常仅导入您指定的文件夹，但当该文件夹中没有
图像且包含子文件夹时，它会自动向下遍历——因此根目录级别的子文件夹以及
根目录 `.daq` 都会被一次性全部导入。 捕获的每个层级都会作为单张图片导入，
其他层级则作为模式提供，而不是每个层级导入一张图片。

直接为**层级子文件夹**命名 （例如 `output/raw/`）同样有效。这样做会保留根目录
`.daq`，因此当您重新从 `raw/` 衍生辐射测量
产品时，请一并复制或指向 DAQ 读取数据——否则时间戳匹配将无法找到对应的文件。

**处理始终从 `raw` 开始。** 在每次采集中，原始帧是处理管道的源文件；
`debayered`、`radiance`、`reflectance` 和 `preview` 作为可视化模式存在，但绝不会被
送回处理管道。重新处理 派生产品将重新应用已经烘焙到其像素中的暗角、CCM和
辐射度运算，因此Chloros会拒绝处理，而非
进行重复处理。有两个值得注意的后果：

- `index/`和`composite/`渲染结果**绝不会**被处理。它们是输出文件，而非捕获文件——
  NDVI的LUT渲染结果在辐射度上没有实质性的解释。
- **未包含** `raw`（例如 `array-capture --processing reflectance`）而导出的捕获文件夹
  没有有效的管道源。这些捕获文件可以正常导入和显示，但 `process` 会跳过
  它们并提示如下：

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  如果您确实需要将派生产品推送通过——例如在
  启用 `demosaic` 状态下捕获的集线器会话， 或旧版文件夹——`--input-level {raw,debayered,processed}`会强制指定
  入口点并覆盖跳过设置。该标志是刻意设计的应急通道；`auto`（默认值）
  绝不会处理没有原始数据的捕获数据。

### 混合滤光片阵列中的跳过捕获

当您在一个阵列中混合使用RGB和多光谱相机时，`array-capture --processing radiance`（或 `reflectance`）会保存多光谱帧并**跳过** RGB 相机——对于宽带传感器而言，每像素的拜耳辐射度没有实际意义。CLI会打印每个已保存的文件（及其导出级别），明确记录每个写入的 `.daq` 文件以及每次跳过操作，因此文件数量并不出人意料：

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

跳过原因标记遵循 `<level>-not-applicable-to-rgb-cam` 的模式。 反射率数据也可通过 `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` 进行跳过，当波段主要位于 DAQ 光传感器辐射计量校准范围之外 （~374–974 nm）——在发货 SKU 中，仅 F988 支持该路径，其支持的工作流为反射率面板工作流。

使用 `--processing debayered` （或 `display`）可包含所有摄像头（无论滤光片类型），或使用默认的 `all` 一次性获取每台摄像头的所有适用级别。

---

## 采集模式、记录器与离线重处理

这些功能均基于**持久数组**运行（请先运行 `array-connect`）。它们与 GUI 采集面板功能一致。

### `array-capture` 模式

`array-capture` 是一条包含四种快门模式及一组导出开关的单一命令：

| 模式 | 标志 | 行为 |
| --- | --- | --- |
| **单次** *(默认)* | (无) | 捕获一个同步的捕获组，然后退出。 |
| **连续** | `--continuous` | 连续进行多次扫描，直至触发 `Ctrl+C`、`--count N` 或 `--duration S`。 |
| **间隔值** | `--interval S` | 每 `S` 秒执行一次扫描（从每次扫描开始计时），边界不变。 |
| **最快** | `--fastest` | 仅原始数据 + 分配的 DAQ 读数 + 组合指数合成数据；跳过辐射度/反射率/显示计算，从而快速生成帧。隐含 `--processing raw --force-daq`。 稍后将保存的 `.daq` 重新处理为校准产品。 |

导出开关（可与任何模式结合使用；均共享同一 GUI/SDK 端点）：

| 标志 | 效果 |
| --- | --- |
| `--processing LEVEL` | 单级导出，或 `all`（默认）。 |
| `--levels L1,L2,…` | 导出类型的显式子集（例如 `raw,radiance,reflectance`）；**覆盖 `--processing`**。 |
| `--aligned` / `--no-aligned` | 将每个成员的非原始导出映射到数组的 [对齐配置文件](#alignment)（共注册）。原始数据保持未变换状态，但会在元数据中携带变换信息。如果数组没有对齐配置文件，则回退为未对齐（并发出警告）。 |
| `--index` / `--no-index` | 保存 / 跳过已配置的每台相机植被指数叠加层。默认：渲染该叠加层。 |
| `--force-daq` | 将分配的DAQ/DLS读数保存为`.daq`旁路文件，即使所选层级并不需要该数据 （例如仅包含原始数据的抓取），以便离线将帧重新处理为反射率/植被指数。 |
| `--smart` | 在触发前等待所有摄像机上的 AE 稳定（参见 [Smart-AE / Smart-Capture](#smart-ae--smart-capture))。 |
| `--compression {deflate,none}` | TIFF 像素压缩。 `deflate`（默认）= 无损 zlib L1 + 水平预测器，每帧全分辨率数据约 4.1 MB；`none` = 未压缩，写入速度约快 5 倍，每帧约 6.3 MB — 当磁盘空间允许时，用于实现最大持续写入速率。两种模式均为无损压缩，导入时显示效果完全一致。 |

> **单次写入TIFF+ 持续速率模型。**捕获数据通过**一次**TIFF文件写入过程完成，该过程包含像素数据 + XMP + IFD0 制造商/型号（在全分辨率 Mono12 模式下测得：压缩格式 36 毫秒 / 6.5 毫秒未压缩，而旧版“先写入再用 ExifTool 重写”方式需约 148 毫秒）；剩余的唯一 ExifTool 处理（EXIF 子 IFD 优化）在异步后台工作线程中运行，即使该处理从未执行，帧也已完成并可直接导入。请注意，DEFLATE 压缩受 Python的GIL，因此压缩写入操作**不会**在各摄像头的写入线程间并行执行——以传感器帧率（~10.4 fps）持续进行8台相机全分辨率拍摄，需要同时具备`--compression none`**和**NVMe级别的磁盘 （~500 MB/s 的持续写入速度）。该参数在 `POST /api/camera/array/capture` 中以 `compression` 的形式提供。

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — 视频/GIF 组合索引（监控级）

记录 **实时组合索引视图**中显示的内容，并将其录制到 `.avi`（以及可选的 `.gif`）中。由于该功能从实时复合信号中提取数据，因此必须打开组合流（例如，该数组正在 GUI 中预览）才能 写入。该功能每 2 秒轮询一次进度，并在遇到 `--duration`、`Ctrl+C` 或录像器自动终止时停止。

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--array-id ID` | 仅数组 | 目标数组（若仅连接一个则省略）。 |
| `-o, --output DIR` | `output` | 输出目录（后端本地）。 |
| `--fps F` | `10` | 录制帧率。 |
| `--duration S` | 直到按下 Ctrl+C | `S` 秒后自动停止。 |
| `--gif` | 关闭 | 同时写入动画 GIF。 |
| `--gif-only` | 关闭 | 仅生成 GIF 文件（不包含 `.avi`）。 |

### `array-burst` — 原始拜耳高帧率连拍（分析级）

直接读取抓取循环的同步组缓冲区 — **无需校准链、无需 exiftool、 无需实时预览**——因此可达到相机的最大抓拍速率。写入原始帧 + 每帧清单 + 每个独立 DLS 读数对应一个 `.daq`（位于 `<output>/bursts/<base>/` 下）。离线重新处理 （下一条命令），或传递 `--build` 以在停止时立即进行处理。

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--array-id ID` | 仅数组 | 目标数组。 |
| `-o, --output DIR` | `output` | 输出目录（突发数据存入 `<DIR>/bursts/<base>/`）。 |
| `--duration S` | 直到按 Ctrl+C | `S` 秒后自动停止。 |
| `--max-frames N` | 无限制 | 处理 `N`个原始帧后自动停止。 |
| `--build` | 关闭 | 停止后，立即重新处理该突发数据（与`array-build-video`相同）。 |
| `--products …` | `combined:index` | 配合 `--build` 使用：指定要构建的视频 （见下文）。 |
| `--fps F` | `10` | 配合 `--build`：输出视频的帧率。 |
| `--save-tiffs` | 关闭 | 使用 `--build`： 同时保存每帧校准后的 TIFF 文件。 |
| `--gif` | 关闭 | 启用 `--build` 时：同时写入动画 GIF 文件。 |

### `array-build-video` — 离线重新处理已保存的连拍数据

将每个原始帧的时间与最近的已保存 `.daq` 读数进行匹配，并将其通过 **与导入管道相同的辐射度/反射率/指数处理链** 进行处理，从而渲染一个或多个视频。

`--products` 是由 `kind:level` 项组成的逗号分隔列表，其中 `kind` ∈ `per_cam` | `combined`，且 `level` ∈ `radiance` | `reflectance` | `index`。一个单独的 `level`（无 `kind:`）默认为 `per_cam`。默认值为 `combined:index`。

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `--burst-dir DIR` | （必填） | 突发文件夹的路径（`…/bursts/<base>/`）。 |
| `--products …` | `combined:index` | `kind:level` 列表， 如上。 |
| `--fps F` | `10` | 输出视频帧率（fps）。 |
| `--save-tiffs` | 关闭 | 同时保存每帧校准后的TIFF文件，与视频。 |
| `--gif` | 关闭 | 同时写入动画 GIF 文件。 |

> **选择合适的录像机。** `array-record` 属于 *监控级* —— 它捕获显示器上实时显示的复合图像，且需要保持数据流打开。`array-burst` → `array-build-video` 属于 *分析级*——它以全速率保存原始传感器数据，并在事后重建经校准的辐射度/反射率/折射率视频，无需实时预览。

### 单色（M3M）单波段相机

**M3M**系列是拜耳**M3C**系列的单色版本的单色版本：每台相机配备一个窄带干涉滤光片（例如 `M3M-<lens>-F<wavelength>`、`M3M-L87-F685`），因此传感器输出的是**单一灰度波段**，不包含拜耳马赛克。 无需进行去马赛克处理，无需消除通道间串扰，也无需设置白平衡——整个RGB显示色彩处理流程在此完全不适用。

这对CLI而言意味着：

- **`lattice white-balance`、`lattice color-profile`、`lattice color`**会检测到单色摄像头，并**跳过该设备并显示一行提示信息**，而不是推送毫无意义的设置。 在同一会话中，它们针对RGB/Bayer M3C相机仍能正常运行。
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** 仍然有效——辐射度和反射度是*按波段*的辐射度图，对于单个波段而言定义非常明确。单色帧携带的是**单位**传感器响应矩阵（无3×3解混操作），因此该平面在校准计算中保持不变。
- **单个单波段相机无法生成植被指数。**NDVI / NDRE 等指数至少需要两个波段（例如 Red + NIR）。若要从单波段硬件中获取指数，需将**多个** M3M 相机对准不同波长，将其对齐为一个多波段堆栈，并针对*该堆栈*计算指数：

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` 符号必须与预设的通道名称**完全**匹配（区分大小写；NDVI&#x27;s 对应小写 `red`、`nir` — 参见 `--list-presets`），且波段侧名称需对应对齐堆栈中的某个波段（离线模式也接受以 0 为起点的波段索引，例如例如 `--channel red=0 --channel nir=1`）。

整个堆栈中的区分标志是模型字符串中的 `M3M` 标记（该标记绝不会出现在 `M3C` 字符串中），并在 GUI/SDK中显示为 `is_mono`。

---

## 主机网卡设置与调优（LATTICE 阵列）

LATTICE 摄像机通过主机的以太网适配器传输 GVSP，因此对于多摄像机阵列，适配器的**驱动程序**和**接收环大小**与链路速率同样重要。错误的设置会在“阵列设置”面板中（以及在`lattice network-analysis` / SDK&#x27;的 `analyze_array_network()` 中），即使摄像头本身运行正常。

### USB 10GbE 适配器 — Realtek RTL8157（“Realtek USB 10GbE 系列控制器”）

| 项目 | 所需值 | 为何重要 |
| --- | --- | --- |
| **驱动程序版本**|**≥ v10.67（2026年1月）**，INF `rtump64x64sta.inf` | 旧版**2016年**驱动程序（v10.65，`rtump64x64.inf`） 在关机/重启/休眠时，会错误处理断电和**`DRIVER_POWER_STATE_FAILURE`（蓝屏 `0x9F`）**相关的错误检查。系统转换过程会卡死（约5分钟超时），用户被迫强制关机， 而反复的非正常关机将**导致 WMI 存储库损坏**（PowerShell/工具开始因 `Invalid class` 报错），并在下次启动时**导致 USB 堆栈卡死**（网卡无法启用；USB 设备停止枚举）。 在依赖干净重启之前，请先从 realtek.com（或 USB 适配器供应商）获取更新。|
| **接收缓冲区**— 关键字 `ReceiveBufferLen` |**256**（驱动程序最大值） | 网卡接收环（RX ring）。驱动程序默认值**32**仅留下约0.26 MB可用环空间——对于多机位突发数据而言远不够用——因此阵列面板会报告`Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB`并阻止连接。当设置为**256**时，环缓冲区容量充足（**实验室 10GbE 主机实测约为 13.5 MB**），为多机位 GVSP 突发传输提供了充足的接收管道余量。（特定配置能否实际 *建立连接* 取决于两项检查——**考虑资源耗尽情况**的准入检查以及**聚合超订阅**检查——而非单纯的突发数据与环带宽的比较；参见 [阵列 fps 与突发模型](#array-fps--burst-model)。) |
| **接收 URB**— 关键字 `PendingReceives` |**64**（最大） | 正在传输中的 USB 请求块；需与接收缓冲区配合使用以吸收突发流量。 |
| **巨型帧** — 关键字 `*JumboPacket` | **9014** | 用于 9000 字节的 GVSP 数据包（每帧数据包数量比 1500 字节帧少 6 倍）。 |

> ⚠️ **网卡驱动程序更新会将这些高级属性重置为默认值。**更新或更换适配器驱动程序后，请**重新应用** `ReceiveBufferLen=256` 和 `PendingReceives=64`，否则即使“硬件未发生任何变化”。这是原本正常运行的设备突然无法连接的首要原因。

请通过**管理员权限** PowerShell 执行（请替换为您的适配器名称，例如 `"Ethernet 5"`）：

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` 适用于 USB 10GbE 适配器。** 现在它会检测网卡类型并调整正确的接收环关键字：对于 PCIe 网卡（如 Intel I219 等），`*ReceiveBuffers`→2048；对于 Realtek **USB** 10GbE 控制器（该控制器不暴露 `*ReceiveBuffers`），`ReceiveBufferLen`→256 + `PendingReceives`→64 用于 Realtek **USB** 10GbE 控制器（该控制器不提供 `*ReceiveBuffers`）。目标值会被限制在各驱动程序报告的最大值（`NumericParameterMaxValue`）之内，因此绝不会写入超出范围的值。请在**提升权限**的终端中运行；与任何基于注册表的调整一样，更改需在适配器重启或系统重启后生效。上述手动 `Set-NetAdapterAdvancedProperty` 命令仍是一个不错的替代方案——它们可实时生效（重新绑定适配器），且无需 重启。

### 网络基础知识（所有 LATTICE 链路）

- **地址配置：** 链路本地地址 `169.254.0.0/16`（GigE Vision LLA）。主机采用静态地址 `169.254.x.x/16`；摄像机和数据采集卡-E 设备在同一范围内自动分配。无需 DHCP/网关。
- **数据包大小：**优先使用巨型数据包（9000），但由自动探测器自动调整——它会在每次连接时重新测量，并且已通过GVSP探测绕过相机1500字节的ICMP上限，因此只要线缆实际支持，它就会采用巨包。仅当您比 ，且建议按命令设置而非永久设置：硬编码会跳过探测，因此如果路径实际上无法承载9000，**每次**捕获都会因`SC_ERR_TIMEOUT -1011`超时（参见[环境变量](#environment-variables)）。
- **RX环带宽随`ReceiveBufferLen`调整：**在默认`32`下，可用环带宽约为0.26 MB（对于任何多摄像头突发传输都太小）；在最大值 `256` 时，其容量较大（在实验室的 10GbE 主机上测得约 13.5 MB），提供了实际的冗余空间。配置能否连接，由考虑数据流耗尽情况的准入检查**以及**下文所述的聚合超额订阅检查——而非单纯的突发速率与环带宽的直接对比。

### 阵列帧率与突发速率模型

如何阅读“阵列设置”面板 （以及 `lattice analyze-array` / SDK 的 `analyze_array_network`）：

- **突发帧数按每台摄像头的实际像素格式分别累加。**单色**M3M**摄像头以**Mono12（2 B/px）**格式传输；**M3C**拜耳摄像头以 8 位或 12 位格式传输（即使请求的是 BayerRG8，TRI032S 也会默认输出 BayerRG12）。因此，一个由 4 台全分辨率摄像头组成的帧大小为**~12.6 MB（若均为8位），但若包含三台12位单色相机则约为~25 MB**。该推算会根据型号（标识缓存）解析每台相机的格式，因此突发数据与实际传输内容相符——而非简单假定为BayerRG8格式。
- **USB以太网适配器的传输速率上限 200 MB/s，无论其标称规格如何。** 将链路速率转换为持续传输速率的效率表源自 PCIe；USB 网卡虽会宣传其 *以太网*链路速率，但受限于USB总线及其驱动程序。一款USB 10GbE适配器曾标称“持续”速率约为1063 MB/s——该数值从未经过实际测试——而由此产生的速率调节导致6–18 %的帧数据损坏，同时仍报告正常的帧率目标值。USB连接的网卡现在被限制在 **200 MB/s**（该限制由总线决定，因此不会随标称速率线性增长；USB 1 GbE适配器可达到约80 MB/s，且不受此影响）。能力记录中的`wire_ceiling_source`以文字形式说明了这一点，而`nic_is_usb` 标志对此进行了标记。无论哪种情况，均可通过 `--wire-ceiling-mbps` 进行覆盖。
- **通量控制考虑了数据流消耗，而非整突发与环形缓冲区的对比。** 同时发生的突发传输只需适应 *瞬态积压* = `max(0, Σ per-cam arrival − host drain) × emit_window`，而非整个突发传输。 在“高速主机/低速 CAM”架构下（例如 **PCIe**10G 主机 + 4× 1 GbE CAM：到达速率 ≈ 320 MB/s，排出速率 ≈ 1063 MB/s） 主机的排空速度快于摄像头的填充速度，积压量 ≈ 0，因此全分辨率模拟发射会**允许**该突发，尽管 25 MB 的突发量超过了 13.5 MB 的环容量。 若将同一组四台摄像头连接到**USB**10GbE 适配器后，数据排出速率为 200 MB/s，而非 1063 ——数据到达速度超过了传输速率，且丢包表现为帧数据损坏，而非帧率降低。在 1 GbE 主机上，摄像头 31.25 MB/s 的 DLThr 下限导致数据到达速度超过传输速率 → 系统会正确地**阻塞**（针对 *此类*阻塞，请缩小ROI或使用≥2的像素合并）。允许通过是**两个**连接门之一——另一个是下文提到的聚合超订阅检查。
- **预测帧率是保守的串行读取上限。**主机抓取循环目前以**串行**（每个摄像头约一个发射窗口），因此该周期受 `max(readout+emit, N × emit)` 限制，且每个摄像头的发射速率被限制在摄像头的**访问链路**带宽内（1 GbE ≈ 80 MB/s），而非主机上行链路。对于 4 台全分辨率 12 位摄像头的阵列，帧率为**~2.8 fps**，与实测的 ~2.7–3.0 帧率相符。帧率被刻意设计为**与曝光时间无关**，因此在光线较暗的场景中，随着曝光时间延长，实际帧率可能会略微低于上限。 串行读取才是真正的帧率限制因素；若对其进行并行化处理，上限将提升至接近单次发射速率。
- **聚合超订阅是连接的硬性阻碍。**每台摄像头的带宽分配下限为**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`)，因此一旦下限被锁定，聚合需求（`per_cam × N`）可能超过**防碰撞的物理链路上限**（`sustained × sim_emit_factor`）。1 GbE 环境下实际全分辨率上限：**1500 MTU时6台摄像头，使用巨包时9台**。该上限仅由链路下限和上限共同决定 ——它**与帧大小无关**，因此**分桶和缩小ROI区域均无济于事**（它们仅降低了每*帧*的字节数，而非由GevSCPD控制的每*秒*字节数）；唯一的解决方法是减少摄像头数量、端到端启用巨型帧， 或使用更快的网卡。其表现为 GVSP 数据包丢失，而非平滑的帧率降低，因此 `analyze-array` 会将可实现帧率清零并输出 `**OVER-SUBSCRIBED**`，而 `array-connect` 在分辨率被锁定时会**拒绝连接**（否则，帧率下调机制会将帧进行分桶处理，但这同样无法清除此类阻塞）。 `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` 将该拒绝降级为针对基准测试的醒目警告——参见 [环境变量](#environment-variables)。

### 数组健康状况——哪个子系统正在丢失帧

已连接数组的 `GET /api/camera/array/<array_id>/capability` 携带一个活跃的
`health` 块， 该状态会在**10秒**的滚动窗口内重新评估。它将帧丢失
细分为两种需要采取相反修复措施的原因，而不是报告一个既不指明原因的“不完整”
率：

| 字段 | 含义 | 涉及子系统 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct`（按串口） | 帧**已到达但结构损坏**— GVSP 数据包丢失。 |**网络**：带宽预算、速率控制、网卡接收环、MTU |
| `never_arrived_rate_pct`（按串口） | 帧**从未到达**——相机未触发，或未发送任何数据。 |**触发/同步**：M8 电缆、`--line`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 每种连接方式下表现最差的摄像机速率。 | — |
| `per_cam_rate_pct` | 每台摄像头的综合不完整率（两种原因合计）。 | — |
| `stable_for_seconds` | 每台摄像头保持在 0.01% 以下的时间长度。 | — |

高于 5% 后端会记录一条名为 `[array-health <id>] WARN` 的日志行，其中注明了分区情况——在
首次违规时、严重性区间发生变化时、该状态持续期间每分钟一次，以及当
该状态消除时记录一次。 损坏的半部分会在每个摄像头首次触发时打印 `[gvsp-corrupt <SN>]` 并
注明原因，随后每 60 秒汇总一次。每次评估结果仍会写入后端日志文件；
无论打印什么内容，计数器都会随着每个缓冲区更新。

同一条记录还会报告整个分配所挂载的编号：

| 字段 | 含义 |
| --- | --- |
| `wire_ceiling_mbps` | 主机当前生效的持续带宽配额，单位为 MB/s。 |
| `wire_ceiling_source` | 该数值来源的文字描述 ——例如 `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` 或 `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | 当 `true` 由 `--wire-ceiling-mbps`（或 GUI 中的 **线缆预算** 字段）将其设为该值时。 |
| `nic_is_usb` | 对于 USB 以太网适配器，`true` —— 参见上文提到的 200 MB/s 上限。 |

**解读：** 非零的 `gvsp_corrupt_rate_pct` 且 `never_arrived_rate_pct` 值为 0
表示触发和电缆同步完美无缺，100% 的丢包发生在网络
路径上 — 降低 `--wire-ceiling-mbps` 值并重新连接。 相反的情况则表明问题出在
同步线缆或触发线上。

> **`--target-fps` 并非导致帧损坏的决定性因素。** GevSCPD 时序控制在
> 连接时仅写入一次，因此降低触发率会改变占空比，而非
> 同时发射的突发速率。 经测量，将需求阈值降低 5 倍未见改善；
> 将线速上限从 240 MB/s 降至 200 MB/s 后，同一测试平台的数据包损坏率从 10.4 %
> 降至 0.00%。

> **TRI032S 固件不支持流中自动缩减。** 运行中的阵列
> 无法自行修复此问题；请断开并重新连接，以便连接时间选择器能
> 根据新的上限重新规划。

### 症状 → 解决方法

| 症状（阵列设置 / 连接 / `analyze_array_network`） | 原因 | 解决方法 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` 被重置为 32（通常发生在驱动程序更新后） | 将 `ReceiveBufferLen`→256，`PendingReceives`→64；重新打开控制面板（如果后端缓存了旧的环大小，则重启后端） |
| 重启/关机卡死；随后出现 `Invalid class` WMI 错误， 网卡无法启用，USB驱动器缺失 | 旧版 2016 Realtek USB 10GbE 驱动程序 → 蓝屏 `0x9F` → 强制关机 | 将适配器驱动程序更新至 ≥ v10.67 (2026)，然后重新应用上述接收环设置 |
| 连接成功但返回低于原生分辨率 | Smart-prep 自动缩小帧以适应线路 | 升级链路 / 接受缩小 / `--force-tier slip-emit-and-capture` |
| 阵列报告的目标帧率正常，但实际交付的帧率仅为其一部分； `health.gvsp_corrupt_rate_pct` 不为零，`never_arrived_rate_pct` 为 0 | 主机推断的带宽预算高于其实际可持续传输的带宽（常见于 USB 以太网适配器、带宽较窄的 PCIe 通道或共享互连结构） | 使用 较低的 `--wire-ceiling-mbps` 值重新连接，并重新检查健康状态块。**非** `--target-fps` — GevSCPD 速率控制在连接时已固定 |
| 已发布组中缺少摄像头；`health.never_arrived_rate_pct` 不为零，`gvsp_corrupt_rate_pct` 为 0 | 触发/同步路径 — 摄像头未触发，非网络问题 | 检查 M8同步线缆及`--line`；确认每个成员均处于待机状态（`TriggerMode=On`） |
| `**OVER-SUBSCRIBED**` / `Wire budget`在 `analyze-array`，或连接被拒绝且已锁定解决方案（`array over-subscribes the wire`） | 每台摄像头的聚合需求 （8 MB/s 下限 × N 台摄像头）超过了防碰撞的安全带宽上限——6 台摄像头以全分辨率通过 1 GbE 传输时 @1500 MTU 时，6 台摄像头全分辨率，9 台使用巨型帧 | 减少摄像头数量、端到端使用巨型帧，或使用更快的网卡。**ROI/像素合并无法解决问题**（上限与帧大小无关）。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` 在测试平台上会覆盖此限制（接受数据包丢失）|

---

## `chloros-cli daq`

Spectral-sensor命令。分为两类：
- **`pool-*`**— 轻量级HTTP客户端，通过后端的持久化池驱动传感器。**这是受支持的路径，也是出厂版CLI中唯一存在的路径。** 后端拥有传输通道，因此图形界面、CLI和SDK脚本均可共享一个活动句柄，而无需争夺串行端口。
- **其他所有情况**（`test`、`record`、`live`、`stream`、 `connect`、`info`、`net`、`ota`、 `sample-rate`, `calibrate`, `serve`, `ws`, `udp`、`mqtt`、`reflectance`、 `login`、`logout`、`status`）——直接硬件访问， 为完整起见，以下将对此进行说明。这些功能需要 `daq` Python 软件包，该软件包**不包含在任何已发布的构建产物中**：编译后的 CLI 排除了该包（`scripts/Build-CLI.ps1` 设置了 `--nofollow-import-to=daq`，而传输层 `pyserial` / `bleak` / `zeroconf` 均不包含该文件），PyPI 上的 SDK 包中也不包含该文件。这些组件仅能通过检出源代码运行，因此请将其视为 MAPIR 内部的开发路径，而非可供外部使用的资源。
- **`discover` / `list`** 介于两者之间：它们是从源代码检出中获取的直接硬件命令，但在已发布的构建中，它们会回退到 `pool-discover`，并由后端执行扫描。 因此扫描功能在任何环境下都能正常工作——这一点至关重要，因为这是获取 DAQ-M 设备 BLE MAC 地址的唯一途径。

> **`chloros-cli daq --help`**（以及 `-h` / `help`）列出了 `pool-*` 的子命令——帮助信息会刻意转发至池客户端，以便反映实际执行的命令。若在已发布的构建中调用 在已发布的构建版本上执行直接硬件子命令，系统会以显式错误退出，明确指出缺失的软件包并引导您返回 `pool-*`；没有任何操作会静默失败。（`discover` / `list` 属于例外——它们会被重定向至 `pool-discover` 并正常运行。）
>
> **客户所需的一切功能均可通过 `pool-*` 访问** ——连接、流式传输、记录经过校准的 `.daq` 文件，以及切换电容曲线。该数据采集设备还可通过 Python 配合 `chloros_sdk.connect_daq_sensor()` 进行控制，该命令使用相同的池化路径。

### DAQ 传感器首次连接工作流

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` 参考

| 子命令 | 用途 |
| --- | --- |
| `daq pool-connect` (智能检测) | 打开后端池中的传感器。 |
| `daq pool-connect --port PORT` | 在特定串行端口上运行 DAQ-U。 |
| `daq pool-connect --ble` | 通过 BLE 连接 DAQ-M，自动扫描 MAC 地址。 |
| `daq pool-connect --mac MAC` | 通过蓝牙低功耗（BLE）连接至已知 MAC 地址的 DAQ-M（隐含 `--ble`）。 |
| `daq pool-connect --eth-host HOST` | 通过以太网连接至已知主机的 DAQ-E。 |
| `daq pool-connect --eth` | 通过以太网进行 DAQ-E，主机自动发现（mDNS + ARP 回退；在 Windows 和 Linux 上，即使 ARP 缓存为空也能正常工作）。 |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | 调整积分窗口/AE状态。 |
| `daq pool-connect --no-stream` | 已连接但尚未开始数据流传输 （通过 `pool-stream --start` 恢复）。 |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | 速率校正配置文件。后端默认值为 `sunshine_cosine`。 |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 扫描每个传输通道，查找可连接的传感器，但不进行连接。**这就是查找 DAQ-M 的 BLE MAC 地址的方法。** `daq discover` / `daq list` 在正式发布版本中会自动路由至此。传感器池中已打开的传感器不会被列出——已连接的 DAQ-M 会停止广播 ——因此请使用 `pool-list` 查询这些传感器。 |
| `daq pool-list` | 显示后端池中的所有传感器。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 释放。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最近的 N 个频谱帧。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 恢复/暂停流式传输。 |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | 开始/停止 .daq 录制。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 运行时切换电容校正配置文件。 |

### 直接硬件子命令（仅限源代码检出 — 正式发布版本中不包含）

> 为完整起见列出。这些命令需要 `daq` Python 软件包以及 `pyserial` / `bleak` / `zeroconf`，这些均未包含在编译好的 CLI 或 PyPI SDK 中——它们仅能通过 MAPIR 源代码检出运行。**如果您使用的是已发布的 Chloros 构建版本，请改用上述 `pool-*` 命令**； 这些命令涵盖连接、流式传输、录制和捕获源的选择。

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

打开、连接到， 并运行已保存的Chloros项目（包含`cameras.json` + `sensors.json` + `project.json`的文件夹）。所有操作均通过后端进行，因此图形界面和CLI生成的硬件状态完全一致。

### 子命令 参考

| 子命令 | 用途 |
| --- | --- |
| `project open PATH` | 打印项目的设备清单（摄像头、阵列、传感器）。 |
| `project devices PATH [--reconnect]` | 列出或重新执行设备发现。 |
| `project connect PATH [--cameras-only] [--sensors-only]` | 连接所有已保存的摄像头/阵列/传感器。 |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 从指定摄像机或阵列进行单帧捕获。 |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 从指定摄像机或阵列进行 N 帧连拍（`-n/--count` 默认值为 5；`-i/--interval` 帧间隔（秒），默认 0）。阵列连拍会去除重复的同步组（过期监视器），因此部分循环的阵列无法返回同一帧的 N 个副本；打印每次迭代的结果。 |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | 通过后端任务将流数据流通过后端任务写入磁盘。`--poll-interval` = `/stats`轮询之间的间隔（秒） （默认 2.0）。 |
| `project sensor read PATH NAME [--json]` | 最新频谱帧。 |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | 记录 .daq 文件。 |
| `project run PATH RECIPE.yaml` | 执行 YAML/JSON 捕获配方。`--dry-run` 仅进行验证而不执行。 |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | 计算数组的对齐参数 — 参见[下方的标志表](#project-align-calibrate-options)。 |
| `project align status PATH NAME [--json]` | 打印当前对齐配置文件。 |
| `project align clear PATH NAME` | 清除缓存的配置文件。 |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | 将一个从属节点的变换。 |
| `project align export PATH NAME --to FILE` | 将配置文件保存至JSON。 |
| `project align import PATH NAME --from FILE [--no-validate]` | 加载已保存的配置文件。 |

#### `project align calibrate` 选项

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | 对齐方法。**这些拼写与 `lattice align-calibrate`** 不同，后者采用简写形式 `orb` / `akaze` / `phase`；在此标志下，这两个命令不可互换。 |
| `--model {translation, rigid, affine, homography}` | `affine` | 变换模型以实现拟合。 |
| `--frames N` | `1` | 将帧快照与平均值同步。 |
| `--reference SN` | 主摄像机 | 参考摄像机序列号；其余所有成员均被变形到该摄像机上。 |
| `--max-features N` | `5000` | ORB特征点数量上限。 |
| `--ratio-threshold F` | `0.75` | Lowe’s比率检验。 |
| `--ransac-threshold-px F` | `3.0` | RANSAC 内点阈值。 |
| `--min-matches N` | `15` | **质量门槛** — 若内点匹配数低于此值，则拒绝该求解结果。 |
| `--max-reproj-err-px F` | `4.0` | **质量阈值** — 若均方根（RMS）重投影误差超过此值，则拒绝该求解结果。 |
| `--checkerboard RxC` | — | `--method checkerboard` 的板级几何信息，例如 `9x6`。 |
| `--name PROFILE` | 空 | 嵌入在已保存的JSON中的轮廓名称。**并非数组名称** — 数组名称为位置参数`NAME`。 |

这两个质量门槛是校准虽能成功求解却仍
拒绝保存的原因：若配置文件未能通过其中任何一个门槛，将导致后续每次
捕获都出现无声的对齐错误，因此会被拒绝而非持久化。

### 示例

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### 配方 DSL

`project run RECIPE.yaml` 支持 YAML 或 JSON 格式文件，用于描述一系列操作：

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

支持的操作：`apply`、`wait`、`capture`、`stream`、`burst`、`sensor`。`burst` 操作需要 `name`（必填）、`count` （默认值 5）、`interval`（秒，默认值 0）、`output`、`format` 以及 `settings`（与- 相机设置形式与 `apply` 相同）；数组连拍使用与 `project burst` 相同的“新同步组”看门狗。

运行命令：

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## 环境变量

| 变量 | 作用 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | 覆盖后端URL（默认值为`http://127.0.0.1:5000`）— **仅由`lattice`、`project` 以及 `daq pool-*` 命令家族才会生效。** 核心命令 （`process`、`login`、`logout`、`status`、`export-status`、`time-sync`、`selftest`) 会将目标设为本地机器并忽略此变量（IPv4 字面量绕过了 Windows `localhost`→`::1` 约每请求 2 秒的惩罚），因此它们始终以本地机器为目标。 |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` 将数组超订阅连接拒绝（每个 CAM 的聚合需求 &gt; 带 `pin_resolution` 的碰撞安全线上限）降级为“发出强烈警告并继续”模式，接受 GVSP 数据包丢失。仅限基准测试使用 — 参见 [阵列 fps 与突发模型](#array-fps--burst-model)）。 |
| `CHLOROS_CLI_MODE` | 由CLI自身设置；指示后端启用并行处理。 |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` 跳过 GVSP 回退探测（仅限 ICMP 结果）。 **这会关闭巨包功能，而不仅仅是减少日志输出**——每条路径上，摄像机仅对长度不超过 1500 的 DF 响应，因此该探测是检测巨包的唯一手段。每次连接可为每台摄像机节省约 1 秒；如果网络*可能* 支持传输巨包。设置此选项时，SDK会发出警告。 |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | 将GVSP数据包大小固定为N字节； 完全跳过探测。建议使用按命令设置（`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`）而非永久设置：固定大小会阻止其适应前端网络， 且在无法承载巨包的路径上将大小固定为9000会导致**每次**捕获都因`SC_ERR_TIMEOUT -1011`而超时。 |
| `TMPDIR`（Linux）| 覆盖 Nuitka 的单文件提取目录。若存在 CLI，则自动使用 `/mnt/ssd/tmp`。 |

---

## 退出代码

| 代码 | 含义 |
| --- | --- |
| `0` | 成功。 |
| `1` | 通用错误（大多数子命令错误）。 |
| `2` | 参数错误。 |
| `130` | 被 Ctrl+C 中断。 |

---

## 故障排除提示

- **“需要登录”** → 在本机上运行一次 `chloros-cli login EMAIL PASSWORD`。
- **“无法连接后端”** → 启动 Chloros 桌面应用，或直接运行后端二进制文件 (`chloros-backend`)，若为远程环境，请检查 `CHLOROS_BACKEND_URL`。
- **`lattice` 命令执行失败，提示“未找到 LATTICE 相机驱动程序”** → 未安装 Arena SDK 运行时； CLI 随 `win32api` 一起发布在 Windows 上，但 C 运行时是 GUI 安装程序的一部分。
- **数组连接/数组设置显示“帧将丢失”或“缩小 ROI 以启用”** → 主机网卡的接收环过小（通常在网卡驱动程序更新后会重置为 32）。请参阅 [主机网卡设置与调优](#host-nic-setup--tuning-lattice-arrays) — 设置 `ReceiveBufferLen=256`、`PendingReceives=64`。
- **系统在重启/关机时卡死，随后出现 WMI `Invalid class` / 网卡无法启用 / USB 驱动器缺失** → 过时的 USB 10GbE 适配器驱动程序导致 `DRIVER_POWER_STATE_FAILURE`（蓝屏 `0x9F`）。更新适配器驱动程序——请参阅 [主机网卡设置与调优](#host-nic-setup--tuning-lattice-arrays)进行更新。
- **Jetson 交换分区警告** → 添加基于文件的交换分区；CLI文件会打印出确切的 `fallocate` / `swapon` 命令。
- **缺少 DAQ 直接命令** → 预期情况：随附的 `chloros-cli` 包已刻意排除 `daq` 包，因此仅包含 `pool-*`（PyPI 上的 SDK 也不包含该包）。请使用 `pool-*`，它通过后端驱动相同的传感器，或者使用来自 Python 的 `chloros_sdk.connect_daq_sensor()`。

---

## 参见

- [Python SDK 参考文档](sdk-reference.md) — 相当于所有 CLI 命令的编程实现。
- [DAQ 传感器指南](../daq/README.md) — 特定传感器的接线与校准。
- 在线文档：`https://mapir.gitbook.io/chloros/cli`
