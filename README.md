<div align="center">

# RLONMUSIC DSP-IR 1200

**Impulse Response Processor · 脉冲响应（IR）在线生成器**

纯 Web 前端 · 13 级 DSP 处理链 · 浏览器端异步分块渲染 · 导出 16/24/32-bit WAV

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Web%20%2F%20No%20Backend-green.svg)]()
[![Version](https://img.shields.io/badge/Version-1200%20v1.0-orange.svg)]()
[![No Build](https://img.shields.io/badge/Build-None%20%28Single%20HTML%29-lightgrey.svg)]()

**中文** | [English](./README.en.md)（待翻译）· 灵感参考：[EchoMusic](https://github.com/hoowhoami/EchoMusic)

</div>

---

## 项目简介

RLONMUSIC DSP-IR 1200 是一个基于**纯 Web 前端架构**的音频 DSP（数字信号处理）与**脉冲响应（Impulse Response, IR）生成工具**。

核心机制：向预设的 **13 级 DSP 处理链**输入标准单位冲激信号（Impulse Signal），在浏览器端**异步分块计算**并渲染导出为高保真的 `.wav` 格式 IR 文件，可直接用于：

- 卷积混响（Convolution Reverb）
- 音箱 / 功放 / 话筒模拟（Cabinet & Amp Simulation）
- 耳机均衡与空间声场补偿（Headphone EQ / Crossfeed）
- 系统频响校准与自定义音色链

**无需后端、无需构建工具、无需安装任何依赖**——一个自包含的 `index.html` 文件，双击即可运行。

## 功能特性

- **13 级 DSP 处理链**，严格顺序、逐级可旁通（Bypass）：
  `Impulse → 9-Band EQ → Compressor → SuperBass → Surround3D → HeadphoneSurround → Clarity → UltrasonicFilter → TubeSim → FDNReverb → PeakNormalize → Watermark → WAV Export`
- **9 段图形均衡器**：RBJ 峰值滤波器（65 Hz–16.7 kHz），±12 dB、Q=1.10 固定、AUTO TRIM 防削波。
- **动态压缩器**：软拐点（Knee）、动态 Attack/Release、AUTO Make-up Gain、TANH 软限制。
- **SuperBass 低频增强**：2 阶分频 + 非线性谐波生成（Natural / Rich 双模式）+ 动态包络跟踪。
- **3D 宽幅声场**（Surround3D）：Schroeder 全通去相关 + 环形延迟 + M/S 宽幅扩展（单声道→立体声）。
- **耳机环绕**（HeadphoneSurround）：CROSSFEED 网络（对侧低通 + 延迟 + 交叉增益）或 128-tap 扩散场 IR 卷积。
- **Clarity 清晰度**：动态高频补偿（电平越低补偿越强，防刺耳）。
- **UltrasonicFilter**：4 阶 Butterworth 低通（20 kHz 截断，防混叠）。
- **TubeSim 胆机模拟**：双级 TANH 非线性饱和 + 非对称偏置 + 干湿混合。
- **8 通道 FDN 混响**：Householder 正交反馈矩阵、素数比延迟线、逐线阻尼（高频衰减）、T60 / Size / Predelay 可调。
- **峰值归一化**：联合峰值归一至 **-0.01 dBFS**（可旁通）。
- **隐形水印**：扩频 LSB（LFSR-32 + BPSK，×64 PN/bit），含长度 + 数据 + XOR 校验的帧结构。
- **标准 RIFF/WAVE 导出**：16-bit PCM / 24-bit PCM / 32-bit Float，含 `LIST INFO`（INAM / ICMT / ISFT）可审计元数据。
- **Web Worker 分块渲染**：`BLOCK=8192` 块处理，进度条 + 状态文本，主线程不卡顿；Worker 不可用时自动同步回退。
- **复古专业机架 UI**：Classic Audio Gear 风格，垂直 EQ 推子、LCD 数值屏、BYPASS 拨杆、信号链芯片联动、12 段 VU 表。
- **结果可视化**：渲染后展示波形（720 桶 min/max 包络）+ 频谱（8192 点 FFT，110 对数分箱）+ PEAK / RMS / 时长统计。

## 快速开始

### 方式一：直接打开（零依赖）

```bash
# 下载后直接双击 index.html 即可在浏览器中运行
```

### 方式二：本地服务器（推荐，更稳定）

```bash
git clone https://github.com/EzRlon/dsp-ir-1200.git
cd dsp-ir-1200
python3 -m http.server 8000
# 浏览器打开 http://localhost:8000
```

## 系统要求

- **浏览器**：Chrome / Edge / Firefox 当前版本（需支持 `Web Worker` 与 `Blob URL`，不支持时自动回退主线程同步渲染）。
- **无需** Node.js / Python / 任何运行时（方式一）。

## 界面预览

**桌面端（1440×900）**

![DSP-IR 1200 桌面端界面](assets/screenshot-desktop.svg)

## DSP 处理链与模块规格

| # | 模块 | 算法要点 | 关键参数 |
|---|------|---------|---------|
| 1 | IMPULSE INPUT | 单位冲激：`x[0]=10^(amp/20)`，其余为 0 | 采样率 44.1/48/88.2/96 kHz；位深 16/24/32；时长 0.25–8 s |
| 2 | GRAPHIC EQ · 9-BAND | RBJ peaking 串联，Q=1.10；AUTO TRIM = `clamp(-mean(|g|)×0.6, ±6) dB` | 65/125/250/500/1K/2K/4K/8K/16.7K Hz，±12 dB |
| 3 | COMPRESSOR | 软拐点包络压缩 + AUTO Makeup + TANH 软限制 | THRESH -60–0 dB；RATIO 1–20:1；ATTACK 0.1–100 ms |
| 4 | SUPER BASS | 2 阶低通/高通分频 + 非线性谐波（tanh）+ 包络跟踪 | X-OVER 40–300 Hz；DRIVE 1–8；NATURAL/RICH |
| 5 | SURROUND 3D | 2× Schroeder 全通去相关 + 环形延迟 + M/S 宽幅 | WIDTH 0–100%；DECORR 1–25 ms；单声道→L/R |
| 6 | HP SURROUND | Crossfeed 网络（1-pole LP + 延迟环）或 128-tap IR 卷积 | MODE：CROSSFEED / IR；AMOUNT 0–100% |
| 7 | CLARITY | 高通分离 + 动态高频补偿增益 | AMOUNT 0–12 dB；THRESH -60–0 dB；FREQ 3–12 kHz |
| 8 | ULTRA FILTER | 2× 二阶 Butterworth = 4 阶低通 | CUTOFF 15–24 kHz（默认 20 kHz） |
| 9 | TUBE SIM | 双级 TANH：`tanh(d·x+b)` → `tanh(v·1.4+b·0.5)·norm` | DRIVE 0.5–20；BIAS ±0.5；MIX 0–100% |
| 10 | FDN REVERB | 8 条素数比延迟线 + Householder 正交反馈 + 逐线阻尼 | PREDELAY 0–250 ms；SIZE 0.25–2.0；T60 0.1–6 s；WET |
| 11 | NORMALIZE | 联合峰值（L/R max）归一至 -0.01 dBFS | TARGET -0.01 dBFS；JOINT PEAK |
| 12 | WATERMARK | LFSR-32 扩频 ×64 chips/bit，BPSK，LSB 级幅度 | MESSAGE ≤24 字符；DEPTH 1–3 LSB |
| 13 | WAV EXPORT | RIFF/WAVE + LIST INFO（INAM/ICMT/ISFT） | 16-bit PCM / 24-bit PCM / 32-bit Float |

## 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│ index.html（单文件自包含）                                    │
│  ├─ <style>  Classic Audio Gear 复古机架样式体系（CSS tokens）│
│  └─ <script type="text/plain" id="dsp-core">                 │
│        DSP 引擎源码（零 DOM 依赖，纯 JS）                     │
│        ├─ Biquad 基类（直接型 II：peaking / lp2 / hp2 / 1-pole）│
│        ├─ Compressor / SuperBass / Surround / Clarity / Tube │
│        ├─ FDN 8 通道混响（Householder 矩阵）                 │
│        └─ WAV 编码器 + 扩频水印 + 归一化                      │
│  └─ 主线程：UI 装配 + Worker 调度 + 进度协议 + 可视化        │
└─────────────────────────────────────────────────────────────┘
        │ Blob URL（dsp-core + WORKER_GLUE）
        ▼
┌─────────────────────────────────────────────────────────────┐
│ Web Worker：renderIR(cfg, onProgress)                        │
│  冲激 → 单声道段（EQ/COMP/BASS，逐块 8192）                  │
│       → 立体声段（SUR3D/HP-SUR/CLAR/ULTRA/TUBE/FDN，逐块）   │
│       → 归一化 → 水印 → WAV 编码 → transferable 回传          │
│  失败/受限时 → 主线程 new Function 同步回退（240s 超时守卫）  │
└─────────────────────────────────────────────────────────────┘
```

### 关键工程约定

- **块处理**：`BLOCK=8192`，末块 `cnt=min(BLOCK, n-off)`；所有滤波器/包络状态**跨块保持**。
- **双段管线**：单声道段（EQ→COMP→BASS）→ 立体声段（SUR3D 为单声道→立体声唯一转折点）。
- **进度协议**：`{type:'progress', done, total}`（total=2×块数），完成回传 `{type:'done', wav, stats, preview}`。
- **缓冲复用**：`tmp/tmp2` 两个 `Float32Array(8192)` 一次性分配复用，避免每块 GC。
- **RIFF 细节**：`riffSize = 36 + dataSize + 8 + info.length`（含 LIST 块头）；INFO 标记用字符码 `[73,78,70,79]`；24-bit 三字节小端、负值 `Math.ceil` 防 -1.0 溢出；32-bit float 不 clamp（允许 >0 dBFS）。
- **元数据**：ICMT 写入可审计的链与参数摘要（`CHAIN:IMPULSE>EQ9>… | 48kHz/24bit | EQ:[gains] | …`）。

## 项目结构

```
dsp-ir-1200/
├── index.html                # 唯一源码文件（自包含，可直接运行）
├── LICENSE                   # GPL-3.0
├── README.md                 # 本文档
└── assets/
    └── screenshot-desktop.svg
```

## 开发与测试

```bash
# 语法校验（DSP 核心 + 主脚本）
# 浏览器打开页面 → F12 控制台零报错

# 建议本地服务器方式开发（避免 file:// 下 Worker 受限）
python3 -m http.server 8000
```

### 验证清单（交付前逐项）

1. WAV 头逐字节断言（RIFF/WAVE、fmt tag、byteRate/blockAlign/bits、dataSize）。
2. 16/24/32-bit 三档位均产出合法 WAV；24-bit 三字节小端还原无符号错误。
3. 归一化开启时输出峰值 ≈ -0.01 dBFS；全旁通时峰值 = 冲激振幅（-6 dB → ≈ -6.02 dBFS）。
4. 水印 LSB 扰动幅度 = `0.9 × 2^(1-bits) × depth`，解调帧自洽。
5. 桌面 / 移动端无横向溢出、无文字重叠。

## 路线图

| 阶段 | 内容 | 状态 |
|------|------|------|
| P1 | 模块化重构 + Web Worker 分块渲染 | ✅ 已完成 |
| P2 | 实时试听预览（Web Audio 节点网络，上传音频实时监听 DSP） | 📋 计划 |
| P3 | 算法精度（FDN 阻尼细化、2x/4x 过采样抗混叠、True Peak） | 📋 计划 |
| P4 | UI 复刻与预设管理（EQ 频响/T60 曲线、JSON 预设导入导出） | 📋 计划 |

## 已知限制

- IR 输出天然包含预延迟（FDN PREDELAY）与各滤波器群延迟，IR 用途下无需延迟补偿。
- 非线性模块（压缩 / 谐波 / 胆机）对单冲激的响应本质上是「稳态非线性近似」，IR 结果以听感 / 频响为准。
- `file://` 下部分浏览器（Firefox 等）Worker 受限，自动回退主线程同步渲染（性能略低，功能一致）。
- 本工具为离线 IR 渲染，**不含**实时流式音频处理（属 P2 计划）。

## 许可证

本项目采用 [GPL-3.0](./LICENSE) 开源协议。

参考与致敬：**[EchoMusic](https://github.com/hoowhoami/EchoMusic)**（GPL-3.0）——其 10 段 EQ、LUFS 标准化与 WAV/IRS 空间音效提供了特性与命名对齐的参考。

---

<div align="center">

**RLONMUSIC DSP-IR 1200 · Impulse Response Processor · Model 1200**

© 2026 EzRlon · Built with pure Web Audio DSP

</div>
