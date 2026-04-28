# ChemTraYzer ReaxFF Post-Processing (Python 3 Port)

> This repository contains a Python 3 port and performance-oriented refactor of a subset of ChemTraYzer scripts for processing LAMMPS `fix reaxff/bond` outputs.

---

## 中文说明

### 1. 项目简介

本仓库包含 3 个核心脚本：

- `processing.py`：读取并处理 LAMMPS `fix reaxff/bond` 输出，生成反应工作文件（`.reac`）。
- `analyzing.py`：对 `.reac` 结果做物种/反应/速率分析并导出表格。
- `log.py`：命令行输出与日志格式化工具。

当前版本重点在：

- Python 2 -> Python 3 兼容迁移；
- OpenBabel 多版本 API 兼容；
- 解析与拓扑更新性能优化；
- 可选并行解析（`-j`）与性能分析输出（`-profile`）。

---

### 2. 安装依赖

推荐 Python 3.9+。核心依赖：

- `openbabel`（或 `openbabel-wheel`）
- `numpy`
- `scipy`

可选依赖（用于绘图/机制图）：

- `matplotlib`
- `pygraphviz`

---

### 3. 基本用法

#### 3.1 处理 ReaxFF bond 文件

```bash
python processing.py bonds.tatb -i 1:6 -i 2:1
```

常用性能参数：

```bash
python processing.py bonds.tatb -i 1:6 -i 2:1 -j 8 -profile
```

- `-j <workers>`：每帧行解析并行 worker 数
- `-profile`：输出分阶段耗时统计

#### 3.2 分析 `.reac` 文件

```bash
python analyzing.py bonds.tatb.reac -main tatb_case
```

---

### 4. 输出说明（简要）

- `*.reac`：按时间步记录反应事件（工作文件）
- `*.spec.tab`：物种统计
- `*.reac.tab`：反应统计
- `*.rate.tab`：速率常数输出（依参数与路径）

---

### 5. 最小可行插件化架构草图（面向 LAMMPS）

目标：把当前 Python 后处理中的热点（extract/split/reduce）下沉到 C++，并保留可验证路径。

#### MVP-Stage 1（最小可行）

1. **Parser Core（C++）**
   - 输入：`fix reaxff/bond` 每帧文本
   - 输出：规范化 bond 列表 + 原子电荷数组
2. **Bond Delta Core（C++）**
   - 输入：前后帧 bond 列表
   - 输出：`depletion/creation`（断键/成键）
3. **Bridge Layer（pybind11/C API）**
   - 暴露给 Python：`parse_frame()`, `reduce_bonds()` 等接口
4. **Python Orchestrator（现有 processing.py）**
   - 保留 `updateSystem/canonical/chkReactions/filterRecrossing` 逻辑
   - 与 C++ 核心拼装，快速迭代验证

#### MVP-Stage 2（进一步加速）

- 评估把 `splitBonds` 与部分拓扑更新（`updateSystem`）一起下沉；
- 与当前 Python 版做逐步回归比对（同一轨迹逐步比较反应事件）。

#### 回归验证建议

- 小体系做逐步对比（每步反应列表完全一致）；
- 大体系做统计对比（总反应数、关键路径、物种曲线）；
- 先保留 Python reference 作为金标准。

---

### 6. 致谢 / Acknowledgement

本仓库基于 **ChemTraYzer** 原始工作进行移植与工程化改造。感谢原始开发者及维护者：

- Malte Doentgen
- Felix Schmalz
- Leif Kroeger
- RWTH Aachen University 相关团队

原始项目页面：

- SourceForge: https://sourceforge.net/projects/chemtrayzer/

---

## English

### 1. Overview

This repository contains three core scripts:

- `processing.py`: parses LAMMPS `fix reaxff/bond` output and writes reaction work files (`.reac`).
- `analyzing.py`: computes species/reaction/rate summaries from `.reac`.
- `log.py`: console/log formatting utilities.

Current focus:

- Python 2 -> Python 3 migration
- OpenBabel API compatibility across versions
- performance optimizations for parsing/topology stages
- optional parallel parsing (`-j`) and runtime profiling (`-profile`)

---

### 2. Dependencies

Recommended: Python 3.9+.

Core:

- `openbabel` (or `openbabel-wheel`)
- `numpy`
- `scipy`

Optional:

- `matplotlib`
- `pygraphviz`

---

### 3. Quick Start

#### 3.1 Process ReaxFF bond file

```bash
python processing.py bonds.tatb -i 1:6 -i 2:1
```

With performance options:

```bash
python processing.py bonds.tatb -i 1:6 -i 2:1 -j 8 -profile
```

#### 3.2 Analyze `.reac`

```bash
python analyzing.py bonds.tatb.reac -main tatb_case
```

---

### 4. Minimal Viable Plugin Architecture (for LAMMPS integration)

#### Stage 1 (MVP)

1. **C++ Parser Core**
   - Input: per-frame `fix reaxff/bond` text
   - Output: normalized bond list + per-atom charges
2. **C++ Bond-Diff Core**
   - Input: old/new bond lists
   - Output: depletion/creation events
3. **Bridge Layer (pybind11 / C API)**
   - Python-callable APIs (`parse_frame()`, `reduce_bonds()`)
4. **Python Orchestrator (existing logic)**
   - keep `updateSystem/canonical/chkReactions/filterRecrossing` in Python first
   - iterate fast while preserving reference behavior

#### Stage 2

- Move more hotspots (e.g., `splitBonds`, partial topology updates) into C++.
- Keep strict regression checks against Python reference implementation.

---

### 5. Credits

This repository is a port/refactor based on original **ChemTraYzer** work.

Thanks to original developers and contributors:

- Malte Doentgen
- Felix Schmalz
- Leif Kroeger
- RWTH Aachen University teams

Original upstream:

- SourceForge: https://sourceforge.net/projects/chemtrayzer/

