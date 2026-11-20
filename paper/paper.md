---
title: "Automating protein structure homology modeling and beta-sheet content prediction for exosome protein samples"
tags:
  - Python
  - homology modeling
  - protein structure prediction
  - batch modeling
  - ProMod3
  - DSSP
  - UniProtKB
  - bioinformatics
authors:
  - name: Troy Timmerman
    affiliation: 1
  - name: Gerardo M. Casanola-Martin
    affiliation: 1
  - name: Rick Jansen
    affiliation: 2
  - name: Dali Sun
    affiliation: 3
  - name: Bakhtiyor Rasulev
    affiliation: 1
    corresponding: true
affiliations:
  - name: Department of Coatings and Polymeric Materials, College of Science and Mathematics, North Dakota State University, Fargo, ND, USA
    index: 1
  - name: Clinical and Translational Science Institute, University of Minnesota, Minneapolis, MN, USA
    index: 2
  - name: Electrical and Computing Engineering, University of Denver, Denver, CO, USA
    index: 3
bibliography: paper.bib
---

# Summary

Protein structure modeling is an important component of biological research, but projects involving hundreds or thousands of proteins require workflows that balance structural quality, computational cost, and throughput. Modern deep-learning approaches have substantially advanced protein structure prediction, while template-based homology modeling remains useful when suitable homologous structures are available and large batches must be processed efficiently [@Baek2021; @Pereira2021].

The **Homology Modeling Pipeline** is a Python-based workflow for automated, local batch homology modeling and secondary-structure analysis. Starting from UniProtKB accession identifiers, the pipeline retrieves protein sequences, identifies homologous templates, obtains and prepares structural coordinates, constructs models using ProMod3, and calculates secondary-structure elements with DSSP [@Bateman2021; @Studer2021; @Kabsch1983]. The resulting structural models and tabulated beta-sheet-related descriptors can be used in downstream statistical and data-driven analyses. The software is intended for Linux workstations and high-performance computing (HPC) environments and enables users to process large protein collections without repeated manual submission to web-based modeling services.

# Statement of need

Deep-learning methods such as RoseTTAFold and related approaches have achieved high accuracy in protein structure prediction [@Baek2021]. Nevertheless, template-based modeling remains valuable when good homologous structures are available, particularly when computational throughput is important. Continuous Automated Model EvaluatiOn (CAMEO) studies have also demonstrated the continuing utility of template-based approaches as reference baselines for structure prediction [@Haas2018; @Haas2019].

Established services such as SWISS-MODEL provide high-quality homology modeling [@Waterhouse2018], but workflows based on interactive web interfaces are inconvenient for projects that require automated processing of hundreds or thousands of protein accessions. Other protein structure prediction servers and methods address related problems [@Kim2004; @McGuffin2019], but a local workflow offers additional control over batch execution, computing resources, intermediate files, and reproducibility.

The Homology Modeling Pipeline addresses this need by integrating sequence retrieval, template identification, template preparation, ProMod3 modeling, and DSSP secondary-structure analysis into a single automated workflow. Its primary use case is high-throughput generation of homology models and structural features, including beta-sheet-related outputs, for protein datasets that would be impractical to process manually.

# State of the field

Established services including SWISS-MODEL, Robetta, and IntFOLD provide protein structure modeling or prediction capabilities [@Waterhouse2018; @Kim2004; @McGuffin2019]. HH-suite, ProMod3, and DSSP address distinct steps in template identification, model construction, and secondary-structure assignment [@Steinegger2019; @Studer2021; @Kabsch1983]. The Homology Modeling Pipeline does not introduce a new prediction algorithm or claim superior structural accuracy. Its contribution is an accession-driven workflow that connects these tools for local, repeatable processing of protein collections and produces model files alongside beta-sheet-related summaries.

The design favors integration rather than modifications to the underlying modeling packages. This choice supports projects that need a consistent sequence-to-secondary-structure workflow and control over computing infrastructure and intermediate files, while retaining the limitations of template-based prediction. It is particularly relevant to exosome-protein datasets, for which many accession identifiers may need to be processed in a consistent way.

# Software design

For each submitted protein accession, the pipeline performs six main operations:

1. **Input parsing:** reads a CSV file containing one UniProtKB accession code per row under the required `Accession` column.
2. **Sequence retrieval:** retrieves the target amino-acid sequence from UniProtKB [@Bateman2021].
3. **Template identification:** queries a local homolog database to identify candidate template structures using HH-suite/HHblits [@Steinegger2019].
4. **Template preparation:** retrieves structural coordinates from the Protein Data Bank and prepares the selected template chains [@Berman2000; @Joosten2011].
5. **Homology model construction:** builds and optimizes the target structure with the ProMod3 modeling engine [@Studer2021].
6. **Secondary-structure quantification:** applies DSSP to calculate secondary-structure elements and summarizes beta-sheet-related outputs for downstream analysis [@Kabsch1983].

The architecture separates sequence retrieval and homology search from template preparation, model generation, and secondary-structure analysis. Reusing established components reduces the need to implement and maintain separate modeling algorithms, but also introduces dependencies on external databases and executables. The workflow prioritizes batch throughput and a consistent tabular output over the breadth of methods available through more general structure-prediction platforms.

The submission repository at [bakhras/pipeline](https://github.com/bakhras/pipeline) is a fork of the original software repository at [troy-lab/pipeline](https://github.com/troy-lab/pipeline). The core pipeline implementation originates in the upstream project; the submission fork preserves that provenance and includes the JOSS manuscript and associated presentation changes. The fork is not presented as an independent reimplementation of the software.

The pipeline is implemented for Python 3.8 or later and has been tested in a Linux environment. Its documented execution environment uses Singularity 3.7 or later together with Git, Wget, HH-suite/HHblits, ProMod3, and DSSP. The software is designed for execution on Linux workstations or HPC clusters and is distributed under the BSD 3-Clause license.

The principal tabulated output is `output.csv`, accompanied by generated protein structure files. For reproducible analyses, users should retain the input accession list, generated models, output tables, software version or Git commit, database build date, container version, operating-system information, and runtime or scheduler logs. Detailed installation instructions, example commands, input preparation, output organization, quality-control recommendations, and troubleshooting guidance are maintained in the repository documentation rather than repeated in this paper.

# Research impact statement

The pipeline was developed for research involving protein collections from exosome samples, where automated preparation of structural models and consistent secondary-structure summaries can reduce manual processing. Its immediate research output is a set of homology models and a table of DSSP-derived descriptors that can be examined or used in subsequent analyses.

The workflow was evaluated as a high-throughput modeling system by processing a batch of 840 proteins on an HPC cluster. The complete batch required approximately 16 hours, corresponding to a mean runtime of approximately 1 minute 14 seconds per protein. Proteins with suitable templates and lengths below approximately 400 amino acids could be modeled in less than 30 seconds, whereas proteins with poorer template candidates or greater sequence lengths generally required longer runtimes.

This evaluation demonstrates the practical scalability of the workflow: hundreds of models and secondary-structure summaries can be generated within a single local batch job rather than through repeated manual web-server submissions. The resulting structural descriptors can subsequently support statistical or machine-learning workflows. The validation is intended to establish batch-processing capability and practical runtime, not to claim that every generated structure is more accurate than predictions from current deep-learning methods.

For applications in which atomic-level structural accuracy is the primary endpoint, users should perform target-specific validation using experimentally determined structures when available, external model-quality metrics, or comparison with alternative structure-prediction methods.

# Limitations

The pipeline depends on the availability and quality of homologous template structures. Proteins with low template coverage, intrinsically disordered regions, unusual domains, or weak homologs may yield unreliable models or no model. Runtime and output quality are also affected by protein length, database completeness, storage performance, and network availability during setup or sequence and template retrieval.

The current workflow targets Linux environments and requires Singularity, so adaptation may be necessary on other operating systems or at institutions without container support. The software is best suited to batch feature generation and secondary-structure analysis and should not be considered a substitute for detailed structural validation when high-resolution atomic accuracy is required.

# AI usage disclosure

ChatGPT (OpenAI) was used for language editing and preparation of draft text addressing the journal's pre-review comments, including the state-of-the-field, design, and impact discussions. The human authors are responsible for independently checking all AI-assisted text against the repository implementation, documentation, and cited sources and for validating all scientific claims and submitted materials.

# Acknowledgements

Research reported in this work was supported by the National Cancer Institute of the National Institutes of Health under award numbers R03CA252783 and R21CA270748. This work also used resources of the Center for Computationally Assisted Science and Technology (CCAST) at North Dakota State University, made possible in part by NSF MRI Award No. 2019077. Additional support was provided by the National Institute of General Medical Sciences of the National Institutes of Health under award U54GM128729.

The authors declare that they have no known competing financial interests or personal relationships that could have appeared to influence the work reported in this paper.

# References
