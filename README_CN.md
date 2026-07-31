# PTDB — 植物转运蛋白数据库

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20593739.svg)](https://doi.org/10.5281/zenodo.20593739)
[![Docker](https://img.shields.io/badge/docker-transporter--pred-blue?logo=docker)](https://hub.docker.com/r/paulfire/transporter-pred)
[![Website](https://img.shields.io/badge/website-PTDB-brightgreen)](https://yanglab.hzau.edu.cn/ptdb/index/home)

一个综合性植物转运蛋白数据库，提供系统分类、进化分析、跨物种比较和在线预测工具。

**网站地址：** [https://yanglab.hzau.edu.cn/ptdb/index/home](https://yanglab.hzau.edu.cn/ptdb/index/home)

## 快速链接

| 资源 | 地址 |
|------|------|
| 网站主页 | https://yanglab.hzau.edu.cn/ptdb/index/home |
| 数据批量下载 | https://yanglab.hzau.edu.cn/ptdb/index/download |
| REST API 文档 | https://yanglab.hzau.edu.cn/ptdb/index/api_documentation |
| 源代码仓库 | https://github.com/Kone1y/PlantTPDB |
| 数据存档（Zenodo） | https://doi.org/10.5281/zenodo.20593739 |
| Docker 镜像 | https://hub.docker.com/r/paulfire/transporter-pred |

## 项目简介

PTDB 整合了来自多个植物物种的转运蛋白信息，主要功能包括：

- **转运蛋白分类**：TC 分类系统、Pfam 结构域、基因家族（ABC、MFS 等）
- **跨物种比较**：共线性分析、系统发育树构建、Ka/Ks 选择压力计算
- **功能注释**：底物搜索、通路映射、文献整合
- **在线工具**：BLAST 序列搜索、转运蛋白预测、基因家族扩张收缩分析

## 分析工具脚本

### 系统发育分析（Phylogenetic Analysis）

基于最大似然法的转运蛋白基因系统发育推断。使用 **MAFFT** 进行多序列比对，再使用 **FastTree (v2.1.11)** 构建系统发育树。用户可选择氨基酸替代模型（JTT / WAG / LG）和位点速率异质性模型（CAT / Gamma）。

```bash
bash Tools/phylogenetic_analysis.sh -i input.fasta -t wag -r gamma -o output/
```

| 参数 | 说明 | 可选值 | 默认值 |
|------|------|--------|--------|
| `-i` | 输入 FASTA 文件 | — | （必填） |
| `-t` | 替代模型 | jtt, wag, lg | jtt |
| `-r` | 速率异质性模型 | cat, gamma | cat |
| `-o` | 输出目录 | — | ./phylogenetic_output |

**依赖工具：** MAFFT、FastTree

**输出文件：**
- `aligned_sequences.fa` — MAFFT 多序列比对结果
- `phylogenetic_tree.nwk` — Newick 格式的最大似然系统发育树

---

### 进化 / 序列相似性分析（Evolution / Sequence Identity）

跨物种同源基因的多序列比对分析。接受多序列 FASTA 文件，运行 **MAFFT** 进行比对，输出 CLUSTAL 格式比对结果及解析后的独立序列。

```bash
bash Tools/evolution_analysis.sh -i input.fasta -o output/
```

| 参数 | 说明 | 可选值 | 默认值 |
|------|------|--------|--------|
| `-i` | 输入 FASTA 文件 | — | （必填） |
| `-o` | 输出目录 | — | ./evolution_output |

**依赖工具：** MAFFT

**输出文件：**
- `alignment.clustal` — CLUSTAL 格式的多序列比对
- `parsed_sequences.fasta` — 解析后的独立序列

---

### 基因家族扩张收缩分析（Gene Family Expansion & Contraction）

使用 **CAFE5** 分析植物物种间基因家族的扩张与收缩。支持两种模式：

**矩阵生成**（同步模式，约 5 分钟）：生成基因家族计数矩阵，不运行完整的扩张收缩分析。

```bash
bash Tools/gene_family_expansion_contraction.sh \
    --species Arabidopsis_thaliana,Oryza_sativa,Glycine_max \
    --matrix-type tc \
    --outdir output/
```

**完整流程**（异步模式，数小时）：运行包含 BUSCO 过滤、IQ-TREE 物种树构建、MCMCtree 年代估算和 CAFE5 扩张收缩分析的完整流程。

```bash
bash Tools/gene_family_expansion_contraction.sh \
    --species Arabidopsis_thaliana,Oryza_sativa,Populus_trichocarpa,Zea_mays \
    --matrix-type tc \
    --outdir output/ \
    --full \
    --email user@example.com
```

| 参数 | 说明 | 可选值 | 默认值 |
|------|------|--------|--------|
| `--species` | 逗号分隔的物种列表（至少 3 个） | — | （必填） |
| `--matrix-type` | 基因家族类型 | tc, symbol | （必填） |
| `--family-list` | 自定义基因家族列表文件 | 文件路径 | （内置列表） |
| `--outdir` | 输出目录 | — | ./cafe_output |
| `--full` | 运行完整异步流程 | — | （关闭） |
| `--email` | 结果通知邮箱 | — | （--full 时必填） |
| `--label` | 自定义任务标签 | — | （自动生成） |

**依赖工具：** planttpdb-cafe（CAFE5 封装工具）；完整模式额外需要 BUSCO、IQ-TREE、MCMCtree (PAML)

**输出文件（矩阵模式）：**
- `results/04_cafe_input/tc.filtered.tsv`（或 `symbol.filtered.tsv`）— 基因家族计数矩阵

---

## 容器化流程（Docker / Singularity）

我们为转运蛋白鉴定流程提供了 Docker 镜像，用户无需手动安装依赖即可复现鉴定与基准测试分析。

```bash
docker pull paulfire/transporter-pred:latest

docker run --rm --platform linux/amd64 \
      -v "$PWD/data/proteins.fasta:/in/input.fa:ro" \
      -v "$PWD/results:/out" \
      -e SPECIES=Oryza_sativa \
      -e PTD_THREADS=10 \
      paulfire/transporter-pred:latest
```

镜像内已打包鉴定流程及其全部依赖和配置文件。镜像标签与数据库发布版本一一对应，因此任一版本都可以在其原始软件环境下重新运行。使用 Singularity/Apptainer 的用户可直接转换镜像：

```bash
apptainer build transporter-pred.sif docker://paulfire/transporter-pred:latest
```

可用标签与运行参数详见 [Docker Hub 页面](https://hub.docker.com/r/paulfire/transporter-pred)。

## 程序化访问（REST API）

除交互式浏览外，PTDB 还提供 REST API，支持脚本调用和高通量查询：

| 功能 | 说明 |
|------|------|
| 基因编号查询 | 获取单个转运蛋白基因的完整注释记录 |
| TC 家族检索 | 获取指定 TC 家族或亚家族的全部成员 |
| 批量查询 | 单次请求提交多个基因编号 |

接口路径、请求/响应格式、访问频率限制及调用示例详见
[https://yanglab.hzau.edu.cn/ptdb/index/api_documentation](https://yanglab.hzau.edu.cn/ptdb/index/api_documentation)。

## 项目结构

```
ptdb/
├── Tools/
│   ├── phylogenetic_analysis.sh             # 系统发育树推断流程
│   ├── evolution_analysis.sh                 # 多序列比对流程
│   └── gene_family_expansion_contraction.sh # CAFE5 基因家族分析流程
├── Readme.md
├── README_CN.md
├── README_Tools.md
├── README_Tools_CN.md
├── CHANGELOG.md
├── LICENSE
├── CITATION.cff
└── ...
```

> 各分析工具的详细文档（数据流程、生物信息学工具及参数、可视化方法）请参见 [README_Tools_CN.md](README_Tools_CN.md)。

## 数据可用性与可重现性

所有核心数据集、预测转运蛋白表格、结构模型、置信度指标、分析脚本和流程配置均存放在稳定的公共存储库中，并提供版本化发布。数据库并非仅支持在线浏览，全部内容均可批量下载。

### 1. 批量下载

所有核心数据集可在 [下载页面](https://yanglab.hzau.edu.cn/ptdb/index/download) 获取，包括：

- 带证据代码与置信度分级的预测转运蛋白表
- TC 分类结果
- 预测的结构与拓扑特征
- AlphaFold 3 结构模型
- 模型质量指标（pLDDT、pTM、PAE 汇总）及结构比对结果

### 2. 程序化访问

提供文档完备的 REST API，支持基因编号查询、TC 家族检索和批量查询，详见上文 [程序化访问（REST API）](#程序化访问rest-api)。

### 3. 源代码与数据存档

- **源代码与配置文件：** [github.com/Kone1y/PlantTPDB](https://github.com/Kone1y/PlantTPDB)
- **数据存档：** 全部核心数据集存放于 Zenodo，概念 DOI 为 [10.5281/zenodo.20593739](https://doi.org/10.5281/zenodo.20593739)。每个发布版本均分配独立的版本 DOI，为每一版数据提供永久、可引用的访问入口。

### 4. 容器化

转运蛋白鉴定流程的 Docker 镜像发布于 [hub.docker.com/r/paulfire/transporter-pred](https://hub.docker.com/r/paulfire/transporter-pred)，可完整复现鉴定与基准测试分析，详见上文 [容器化流程](#容器化流程docker--singularity)。

### 5. 版本化发布

- 主版本大致按年度周期发布。
- 每个主版本均附带更新日志（changelog）和可引用的 Zenodo 存档快照。
- 主版本之间，文献追踪模块以 7 天为周期更新知识库，每次更新均记入版本历史。
- 本仓库的 Git 标签与数据库发布版本一一对应。

### 6. 长期维护

PTDB 由华中农业大学托管与维护，**保障运行至 2035 年及以后**。同时在主站之外独立维护镜像存档存储，即使网站服务中断，数据仍可获取。问题反馈与数据需求在本仓库中公开跟踪。

## 引用

如果您在研究中使用了 PTDB，请按以下格式引用：

```
Liang, G., Huang, W., & Luo, C. (2026). PTDB: Plant Transporter Database.
Zenodo. https://doi.org/10.5281/zenodo.20593739
```

如需引用特定版本的数据集，请使用对应 [Zenodo 记录](https://doi.org/10.5281/zenodo.20593739) 中列出的版本 DOI。

## 许可证

本项目按照 LICENSE 文件中指定的条款发布。

## 联系方式

如有问题、Bug 报告或数据需求，请在本仓库中提交 Issue。
