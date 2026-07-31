# Changelog

All notable changes to PTDB will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Release policy: major releases follow an approximately annual cycle, each accompanied by
a changelog entry and a citable Zenodo snapshot. The literature-tracking module updates
the knowledge base on a 7-day cycle between major releases; interim updates are recorded
in the version history on the home page.

## [1.1.0] - 2026-06-27

Revision release addressing peer-review feedback. Adds experimental structure
integration, quantitative confidence and benchmarking metrics, evidence-grounded
question answering, batch/programmatic access, and reproducibility infrastructure.

### Added

Experimental structures
- Integration of experimentally determined structures retrieved from the RCSB PDB
  (333 non-redundant entries), mapped to PTDB proteins by BLASTP against the PDB
  SEQRES database (E-value <= 1e-10, sequence identity >= 30%, query coverage >= 60%)
- "Search by PDB" browsing page listing PDB ID, experimental method, resolution,
  matched chain, bound ligands, and a direct RCSB link
- "Experimental Structures" panel on protein information pages, showing the mapped
  experimental structure side by side with the AlphaFold 3 model in an embedded
  Mol* Viewer (21,440 of 59,494 modelled transporters have a qualifying match;
  the remainder are marked "No matching PDB ID")
- Per-entry mapping quality on protein information pages (sequence identity, query
  coverage, alignment score), labelled "exact-match" or "homologous-template"
  reciprocal-best-match predicted-experimental structure pairs

Structural confidence metrics
- Per-model confidence metrics (pLDDT, pTM, PAE) displayed for every AlphaFold 3
  model and available for single-entry and bulk download
- Per-residue pLDDT values and low-confidence-region statistics by secondary-structure
  class (DSSP-based) for all 59,494 models
- Explicit scope and limitation statements on the structure module: models are
  monomeric predictions without ligands, cofactors, ions, or membrane context, and
  transmembrane orientation is not inferred from them

Knowledge integration
- Evidence-grounded answer generation in the PlantTP Agent: factual statements linked
  to source publications through clickable DOI citations, gene and protein mentions
  linked to their PTDB data cards, and explicit "Experimental" vs "Predicted" labels
  for structural evidence
- "Sources & Evidence" panel on every answer, listing references (with DOIs), database
  entries consulted, structural files referenced, and statements that are inferential
  rather than directly supported
- Tiered confidence labels per statement (literature-grounded, database-grounded, or
  system inference)
- Explicit "insufficient evidence" response when retrieved evidence is inadequate,
  plus a pre-retrieval gating model that validates query scope, requested data type,
  and identifier format
- Community-feedback function on the PlantTPKG page allowing users to flag questionable
  entities or relations for expert review

Data provenance and quality
- Evidence code (DB / PF / TC) and confidence tier ("High" for all three criteria,
  "Medium" for any two) on every retained entry, displayed and filterable on the
  "Search by Index" page
- Evidence-source labelling for subcellular localization: experimentally validated
  annotations from SUBA and cropPAL shown in a dedicated column, separately from
  DeepLoc 2.0 predictions
- Low-completeness flags on the Species page for proteomes with BUSCO completeness
  below 90% (retained for browsing, excluded from comparative family-evolution analyses)
- Species-level proteome metrics: source database, version, BUSCO completeness, total
  protein count, transporter gene count, and transporter isoform count
- ABC-family notice warning of a relatively high missed-detection rate, and
  evidence-strength notices for the 40 families from TC classes 4-9

Analysis modules
- Interactive TC Family Expansion & Contraction module based on CAFE5
- User-selectable amino-acid substitution models (JTT / WAG / LG) and site-rate
  heterogeneity models (CAT / Gamma) in the Phylogenetic module, with a notice that
  alignment (MAFFT) and ML inference (FastTree) are fixed and that Bayesian and
  distance-based methods are not supported
- Batch query support in the "Search by Transporter" and "Search by Index" modules
- Multi-sequence FASTA input for the online Prediction and BLAST modules
- Click-through example queries and built example pages for BLAST

Access and infrastructure
- RESTful API documentation page with parameter reference and usage examples for
  identifier-based, TC-family-based, and batch queries
- Pipeline source code and configuration files published on GitHub
  (https://github.com/Kone1y/PlantTPDB)
- Versioned archival deposition of all core datasets on Zenodo
  (DOI: 10.5281/zenodo.20593739)
- Docker image for reproducing the transporter-identification pipeline
  (https://hub.docker.com/r/paulfire/transporter-pred)
- Versioned-release and changelog module on the home page recording per-release changes
- Long-term maintenance statement: hosted and maintained by Huazhong Agricultural
  University through at least 2035, with mirrored archival storage maintained
  independently of the primary web portal

### Changed
- Home page totals relabelled as transporter genes and protein isoforms
- Literature knowledge base restructured: 19,703 screened publications, 10,160 curated
  core articles, PlantTPKG built from 2,220 curated articles

### Fixed
- Corrected the database-wide aggregate totals to 1,993,520 predicted transporter
  protein isoforms encoded by 1,728,695 genes; the previous figure of 2,012,081
  isoforms included some species counts more than once. Species-level counts are
  unchanged and no downstream analysis is affected.
- Broken release link in this changelog now points to the correct repository

## [1.0.0] - 2026-06-08

### Added
- Initial public release of PTDB (Plant Transporter Database)
- Web portal with transporter classification, search, and visualization
- Multi-species transporter data integration across plant species
- Gene family browser (ABC, MFS, OPT, etc.)
- Pfam domain classification and TC (Transporter Classification) system browser
- Phylogenetic tree visualization based on FastTree
- Synteny analysis for cross-species collinear gene blocks
- Ka/Ks selective pressure analysis
- Pathway mapping for transporter-metabolite associations
- BLAST search against the PTDB dataset via SequenceServer
- Online transporter prediction tool (TC classification, Pfam, TMHMM, DeepLoc, SignalP)
- AI Agent for intelligent natural language queries
- Substrate search and literature integration
- Interactive data visualization using Highcharts and ECharts
- Bulk data download page
- RESTful API for programmatic data access
- Multi-sequence alignment via MAFFT

### Data
- Core transporter annotation datasets
- Predicted transporter classification tables
- Gene family assignments
- Phylogenetic tree files (Newick format)
- Structural model confidence metrics (AlphaFold pLDDT/pTM)
- PDB homology alignment results with structure reference scores
- Literature reference collection

[1.1.0]: https://github.com/Kone1y/PlantTPDB/releases/tag/v1.1.0
[1.0.0]: https://github.com/Kone1y/PlantTPDB/releases/tag/v1.0.0
