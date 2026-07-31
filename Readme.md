# PTDB — Plant Transporter Database

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20593739.svg)](https://doi.org/10.5281/zenodo.20593739)
[![Docker](https://img.shields.io/badge/docker-transporter--pred-blue?logo=docker)](https://hub.docker.com/r/paulfire/transporter-pred)
[![Website](https://img.shields.io/badge/website-PTDB-brightgreen)](https://yanglab.hzau.edu.cn/ptdb/index/home)

A comprehensive web-based database for plant transporter proteins, providing systematic classification, evolutionary analysis, cross-species comparison, and online prediction tools.

**Website:** [https://yanglab.hzau.edu.cn/ptdb/index/home](https://yanglab.hzau.edu.cn/ptdb/index/home)

## Quick Links

| Resource | Location |
|----------|----------|
| Web portal | https://yanglab.hzau.edu.cn/ptdb/index/home |
| Bulk download | https://yanglab.hzau.edu.cn/ptdb/index/download |
| REST API documentation | https://yanglab.hzau.edu.cn/ptdb/index/api_documentation |
| Source code | https://github.com/Kone1y/PlantTPDB |
| Archived datasets (Zenodo) | https://doi.org/10.5281/zenodo.20593739 |
| Docker image | https://hub.docker.com/r/paulfire/transporter-pred |

## Overview

PTDB integrates transporter protein information from multiple plant species, offering:

- **Transporter Classification**: TC system, Pfam domain, Gene Family (ABC, MFS, etc.)
- **Cross-Species Comparison**: Synteny analysis, phylogenetic trees, Ka/Ks calculation
- **Functional Annotation**: Substrate search, pathway mapping, literature integration
- **Online Tools**: BLAST search, transporter prediction, gene family expansion & contraction analysis

## Analysis Tools

### Phylogenetic Analysis

Maximum-likelihood phylogenetic inference for transporter genes. The pipeline performs multiple sequence alignment with **MAFFT**, followed by tree construction with **FastTree (v2.1.11)**. Users can select the amino-acid substitution model (JTT / WAG / LG) and site-rate heterogeneity model (CAT / Gamma).

```bash
bash Tools/phylogenetic_analysis.sh -i input.fasta -t wag -r gamma -o output/
```

| Parameter | Description | Options | Default |
|-----------|-------------|---------|---------|
| `-i` | Input FASTA file | — | (required) |
| `-t` | Substitution model | jtt, wag, lg | jtt |
| `-r` | Rate heterogeneity | cat, gamma | cat |
| `-o` | Output directory | — | ./phylogenetic_output |

**Dependencies:** MAFFT, FastTree

**Output:**
- `aligned_sequences.fa` — MAFFT multiple sequence alignment
- `phylogenetic_tree.nwk` — Newick-format ML phylogenetic tree

---

### Evolution / Sequence Identity

Homologous gene comparison across species with multiple sequence alignment. Accepts a multi-sequence FASTA file, runs **MAFFT** for alignment, and outputs the result in CLUSTAL format along with parsed individual sequences.

```bash
bash Tools/evolution_analysis.sh -i input.fasta -o output/
```

| Parameter | Description | Options | Default |
|-----------|-------------|---------|---------|
| `-i` | Input FASTA file | — | (required) |
| `-o` | Output directory | — | ./evolution_output |

**Dependencies:** MAFFT

**Output:**
- `alignment.clustal` — CLUSTAL-format multiple sequence alignment
- `parsed_sequences.fasta` — Individual parsed sequences

---

### Gene Family Expansion & Contraction

Analysis of gene family gain and loss across plant species using **CAFE5**. Supports two modes:

**Matrix generation** (synchronous, ~5 minutes): Generates a gene family count matrix without running the full expansion/contraction analysis.

```bash
bash Tools/gene_family_expansion_contraction.sh \
    --species Arabidopsis_thaliana,Oryza_sativa,Glycine_max \
    --matrix-type tc \
    --outdir output/
```

**Full pipeline** (asynchronous, several hours): Runs the complete analysis including BUSCO filtering, IQ-TREE species tree construction, MCMCtree divergence time estimation, and CAFE5 expansion/contraction analysis.

```bash
bash Tools/gene_family_expansion_contraction.sh \
    --species Arabidopsis_thaliana,Oryza_sativa,Populus_trichocarpa,Zea_mays \
    --matrix-type tc \
    --outdir output/ \
    --full \
    --email user@example.com
```

| Parameter | Description | Options | Default |
|-----------|-------------|---------|---------|
| `--species` | Comma-separated species list (min 3) | — | (required) |
| `--matrix-type` | Gene family type | tc, symbol | (required) |
| `--family-list` | Custom gene family list file | file path | (built-in list) |
| `--outdir` | Output directory | — | ./cafe_output |
| `--full` | Run full async pipeline | — | (off) |
| `--email` | Email for notification | — | (required with --full) |
| `--label` | Custom job label | — | (auto-generated) |

**Dependencies:** planttpdb-cafe (CAFE5 wrapper); full mode additionally requires BUSCO, IQ-TREE, MCMCtree (PAML)

**Output (matrix mode):**
- `results/04_cafe_input/tc.filtered.tsv` (or `symbol.filtered.tsv`) — Gene family count matrix

---

## Containerized Pipeline

A Docker image of the transporter identification pipeline is provided so that the identification and benchmarking analyses can be reproduced without manual dependency installation.

```bash
docker pull paulfire/transporter-pred:latest

docker run --rm --platform linux/amd64 \
      -v "$PWD/data/proteins.fasta:/in/input.fa:ro" \
      -v "$PWD/results:/out" \
      -e SPECIES=Oryza_sativa \
      -e PTD_THREADS=10 \
      paulfire/transporter-pred:latest
```

The image bundles the identification workflow together with its dependencies and configuration files. Image tags follow the database release versions, so a given release can always be re-run against the exact software stack used to produce it. Singularity/Apptainer users can convert the image directly:

```bash
apptainer build transporter-pred.sif docker://paulfire/transporter-pred:latest
```

See the [Docker Hub page](https://hub.docker.com/r/paulfire/transporter-pred) for available tags and runtime options.

## Programmatic Access (REST API)

In addition to interactive browsing, PTDB exposes a REST API for scripted and high-throughput use:

| Capability | Description |
|------------|-------------|
| Gene-identifier lookup | Retrieve the full annotation record for a single transporter gene |
| TC-family retrieval | Retrieve all members of a given TC family or subfamily |
| Batch queries | Submit multiple identifiers in a single request |

Endpoint paths, request/response schemas, rate limits, and worked examples are documented at
[https://yanglab.hzau.edu.cn/ptdb/index/api_documentation](https://yanglab.hzau.edu.cn/ptdb/index/api_documentation).

## Project Structure

```
ptdb/
├── Tools/
│   ├── phylogenetic_analysis.sh             # Phylogenetic tree inference pipeline
│   ├── evolution_analysis.sh                 # Multiple sequence alignment pipeline
│   └── gene_family_expansion_contraction.sh # CAFE5 gene family analysis pipeline
├── Readme.md
├── README_CN.md
├── README_Tools.md
├── README_Tools_CN.md
├── CHANGELOG.md
├── LICENSE
├── CITATION.cff
└── ...
```

> For detailed documentation of each tool's data flow, bioinformatics tools, and parameters, see [README_Tools.md](README_Tools.md).

## Data Availability & Reproducibility

All core datasets, predicted transporter tables, structural models, confidence metrics, analysis scripts, and pipeline configurations are deposited in stable public repositories with versioned releases. The database is not browse-only: the complete contents can be downloaded in bulk.

### 1. Bulk download

All core datasets are available at the [Download page](https://yanglab.hzau.edu.cn/ptdb/index/download), including:

- Predicted transporter tables with evidence codes and confidence tiers
- TC classification results
- Predicted structural and topological features
- AlphaFold 3 structural models
- Model-quality metrics (pLDDT, pTM, PAE summaries) and structural-comparison results

### 2. Programmatic access

A documented REST API supports gene-identifier lookup, TC-family retrieval, and batch queries — see [Programmatic Access](#programmatic-access-rest-api) above.

### 3. Source code and archival datasets

- **Source code and configuration files:** [github.com/Kone1y/PlantTPDB](https://github.com/Kone1y/PlantTPDB)
- **Archived datasets:** deposited on Zenodo under the concept DOI [10.5281/zenodo.20593739](https://doi.org/10.5281/zenodo.20593739). Each released dataset version receives its own version-specific DOI, providing permanent and citable access to the exact data underlying any given analysis.

### 4. Containerization

Docker containers for the identification pipeline are published at [hub.docker.com/r/paulfire/transporter-pred](https://hub.docker.com/r/paulfire/transporter-pred), enabling full computational reproducibility of the identification and benchmarking analyses. See [Containerized Pipeline](#containerized-pipeline) above.

### 5. Versioned releases

- Major releases follow an approximately annual cycle.
- Each major release ships with a changelog entry and a citable Zenodo snapshot.
- Between major releases, the literature-tracking module refreshes the knowledge base on a 7-day cycle; every update is recorded in the version history.
- Git tags in this repository correspond one-to-one with released database versions.

### 6. Long-term maintenance

PTDB is hosted and maintained by Huazhong Agricultural University through **at least 2035**. Mirrored archival storage is maintained independently of the primary web portal, so that datasets remain retrievable even if the web service is interrupted. Issues and data requests are tracked publicly in this repository.

## Citation

If you use PTDB in your research, please cite it as follows:

```
Liang, G., Huang, W., & Luo, C. (2026). PTDB: Plant Transporter Database.
Zenodo. https://doi.org/10.5281/zenodo.20593739
```

To cite a specific dataset version, use the version-specific DOI listed on the corresponding [Zenodo record](https://doi.org/10.5281/zenodo.20593739).

## License

This project is released under the terms specified in the LICENSE file.

## Contact

For questions, bug reports, or data requests, please open an issue in this repository.
