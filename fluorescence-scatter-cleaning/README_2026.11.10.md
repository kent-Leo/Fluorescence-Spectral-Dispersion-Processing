# Fluorescence Scatter Cleaning

**荧光光谱连续去杂峰** · `fluorescence-scatter-cleaning`

[中文](#中文) · [English](#english)

## 中文

### 简介

本 Skill 用于普通荧光发射光谱和三维激发–发射矩阵（EEM）的散射杂峰处理。在保留原始数据和采样网格的基础上，确认散射区域，采用局部线性插值或保形三次插值（PCHIP）连续补点，生成便于绘图、复查和复现的处理副本。

每个补点都与实测值区分，并记录原值、支撑点和计算方法。与主发光峰重叠的区域单独标记，估计结果仅供绘图展示。

### 安装与调用

1. 解压后，将整个 `fluorescence-scatter-cleaning` 文件夹放入运行环境的技能目录，保持内部结构完整。
2. 当前[官方 Codex 文档](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)列出的本地位置包括用户级 `~/.agents/skills/` 和项目级 `.agents/skills/`。若环境已从其他配置目录加载本 Skill，继续使用现有位置，避免重复安装同名副本。
3. 提供原始数据及可用的测量参数，在支持 `$` 技能调用的 Codex 环境中使用下列请求；其他客户端通过其技能选择器调用。加载方式见[官方说明](https://learn.chatgpt.com/docs/build-skills#how-chatgpt-and-codex-use-skills)。

```text
使用 $fluorescence-scatter-cleaning 处理这份荧光光谱：
仅去除可确认的散射峰，连续补点且不留空白；
保留原始数据，标记所有估计值，并提供逐点处理记录。
```

建议同时说明是否有同条件溶剂空白、需要处理的工作表，以及重叠区是否接受仅供绘图的估计。已经在当前对话中确认的信息会继续沿用。

### 处理流程

1. **核对数据。** 确认 EX/EM 轴向、单位、采样间隔、实际扫描范围和数据维度，保留原始文件。
2. **确认散射。** 结合峰位随激发波长的变化、峰宽、仪器条件及可用空白确定处理范围，保留无法确认的信号。
3. **局部补点。** 普通发射光谱使用有双侧实测支撑的线性插值或 PCHIP；EEM 根据发射带形状与支撑条件选择沿 EX 或 EM 内插。
4. **处理特殊区域。** 主峰重叠或边界缺少双侧支撑时，明确处理选择后再估计，并单独标记。
5. **核验并导出。** 检查数值完整性、带外数据一致性、支撑点、补点计算和曲线衔接，保存后重新读取复核。

### 输出内容

- 保留原始数据及参数的处理副本。
- 维持原采样网格、补点位置清晰标记的绘图数据。
- 逐点处理记录，包括坐标、原值、补点值、方法及支撑数据；线性插值附权重，PCHIP 附结点信息。
- 处理范围、补点数量、方法说明和处理前后对比图。

文件名称带日期，采用 `YYYY.M.D` 格式，例如 `Br_去杂峰_连续填补_2026.11.10.xlsx`。工作表数量和布局随任务调整。

### 数据使用原则

散射范围需要根据每批数据重新判断。没有匹配空白时，保留无法确认的拉曼及其他信号；负值和散射范围外的实测值保持原样。默认不进行全谱平滑、基线归零、归一化或仪器响应校正。

插值值默认用于绘图。主峰被散射遮盖时，现有数据不足以确定真实峰高、峰位或面积；定量分析需使用有效实测值并排除散射干扰。网格完整与曲线相连，是数据完整性检查的一部分，不能代替光谱恢复可靠性的判断。

### 运行环境与案例

本包提供工作流程、界面元数据和案例说明，计算与绘图由执行环境提供。环境需要具备表格读写、数值插值和科学绘图能力；采用 Python 时，可使用 SciPy 实现 PCHIP。具体文件格式的读取能力取决于运行环境。

[Br 案例](references/br-case-2026.11.10.md)记录了 21 条发射光谱及一张 39×71 EEM 的处理，包括 EX 260 nm 主峰重叠区的估计方法。案例参数和处理数量只用于复现该批数据，不作为其他样品的默认设置。

## English

### Overview

This skill guides scatter removal in fluorescence emission spectra and excitation–emission matrices (EEMs). It identifies supported scatter regions and fills them using local linear interpolation or the piecewise cubic Hermite interpolating polynomial (PCHIP), preserving the original sampling grid and measurements outside those regions.

The output distinguishes measured values from estimates and records the original values, supporting measurements, and calculation method. Estimates in regions where scatter overlaps the fluorescence peak are marked separately and used for plotting only.

### Installation and use

1. Extract the archive and place the entire `fluorescence-scatter-cleaning` folder in your host's skills directory. Preserve its internal structure.
2. The current [official Codex documentation](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills) lists `~/.agents/skills/` for user skills and `.agents/skills/` for project skills. If your environment already loads this skill from another configured directory, keep that location and avoid duplicate installations with the same name.
3. Provide the raw data and available measurement parameters. In a Codex environment supporting `$` skill invocation, use the prompt below. Other clients may provide a skill selector; see the [official invocation guide](https://learn.chatgpt.com/docs/build-skills#how-chatgpt-and-codex-use-skills).

```text
Use $fluorescence-scatter-cleaning to process these fluorescence spectra.
Remove only confirmed scatter peaks and fill the affected regions without
leaving blank values. Preserve the raw data, mark all estimates, and provide
a point-by-point processing log.
```

Include whether a solvent blank measured under matching conditions is available, which worksheets to process, and whether plotting estimates are acceptable in overlapping regions. Decisions already confirmed in the current conversation remain applicable to that task.

### Workflow

1. **Inspect the data.** Verify EX/EM orientation, units, sampling intervals, actual scan limits, and dimensions. Preserve the source file.
2. **Confirm scatter regions.** Use peak movement across excitation wavelengths, peak widths, instrument conditions, and available blanks. Retain signals whose origin remains unresolved.
3. **Interpolate locally.** Use linear interpolation or PCHIP with measured support on both sides of an emission-spectrum region. For EEMs, choose the EX or EM direction based on the emission profile and available support.
4. **Handle special regions.** Establish the approach for fluorescence-peak overlap or scan boundaries lacking two-sided support before estimating values. Mark these estimates separately.
5. **Verify and export.** Check numerical completeness, unchanged measurements outside the masks, support points, interpolation calculations, and joins. Reopen the exported file for verification.

### Outputs

- A processed copy retaining the raw data and measurement parameters.
- Plotting data on the original grid, with estimated positions clearly marked.
- A point-by-point log containing coordinates, original and estimated values, methods, and supporting data; linear interpolation includes weights, while PCHIP includes knot information.
- Processing ranges, replacement counts, method notes, and before-and-after plots.

Output filenames include a date in `YYYY.M.D` format, such as `Br_scatter_cleaned_2026.11.10.xlsx`. Worksheet count and layout depend on the task.

### Data-use principles

Determine scatter ranges separately for each dataset. Without a matching blank, retain unconfirmed Raman and other signals. Preserve negative values and measured values outside confirmed scatter regions. Whole-spectrum smoothing, baseline zeroing, normalization, and instrument-response correction are not default steps.

Interpolated values are intended for plotting by default. When scatter obscures a fluorescence peak, the available data do not establish its true height, position, or area. Quantitative analysis requires valid measured values with scatter interference excluded. A complete grid and connected curve demonstrate numerical completeness, not reliable recovery of the underlying fluorescence.

### Runtime and case study

This package contains workflow instructions, interface metadata, and a case reference. The host supplies the computation and plotting capabilities, including spreadsheet access, interpolation, and scientific visualization. A Python implementation can use SciPy for PCHIP. Supported input formats depend on the host environment.

The [Br case study](references/br-case-2026.11.10.md) documents 21 emission spectra and a 39×71 EEM, including plotting estimates where scatter overlaps the fluorescence peak at EX 260 nm. Its ranges and processing counts are reproduction targets for that dataset, not defaults for other samples.

## Package contents / 文件结构

```text
fluorescence-scatter-cleaning/
├── SKILL.md
├── README_2026.11.10.md
├── agents/
│   └── openai.yaml
└── references/
    └── br-case-2026.11.10.md
```

- [SKILL.md](SKILL.md)：完整执行流程 / Full workflow instructions.
- [agents/openai.yaml](agents/openai.yaml)：名称、简介与调用提示 / Display metadata and invocation prompt.
- [Br case / Br 案例](references/br-case-2026.11.10.md)：本批数据的方法与参数 / Dataset-specific methods and parameters.

## Method references / 方法资料

- [Bahram et al., Handling of Rayleigh and Raman scatter for PARAFAC modeling of fluorescence data using interpolation](https://doi.org/10.1002/cem.978).
- [SciPy: PchipInterpolator](https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.PchipInterpolator.html).
- [Edinburgh Instruments: Second Order Diffraction through a Monochromator](https://www.edinst.com/resource/second-order-diffraction-through-a-monochromator-common-errors-in-fluorescence-spectroscopy/).

这些资料提供方法背景；本案例的散射范围和估计锚点根据该批数据设定。 / These sources provide methodological background; the case-specific scatter ranges and estimated knots were selected for that dataset.
