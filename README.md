# rHetBin1.2

This repository contains the workflows that were used to assemble the genome
rHetBin1.2 for *Heteronotia binoei*.

The repo was produced automatically from boilerplate code at
[AToL-Bioinformatics/genome-launcher-workflow](https://github.com/AToL-Bioinformatics/genome-launcher-workflow).

### Assembly data

<details>

<summary>Click to view YAML</summary>

```yaml
assembly_version: 2
augustus_dataset_name: Xenopus_tropicalis
busco_odb10_dataset_name: sauropsida
busco_odb12_dataset_name: squamata
dataset_id: rHetBin1
hic_motif: GATC,GANTC,CTNAG,TTAA
mitochondrial_genetic_code_id: 2
mitohifi_reference_species: Heteronotia binoei
ncbi_class: Lepidosauria
oatk_hmm_name: lepidosauria_mito
read_files:
- data_type: PACBIO_SMRT
  name: bpa-ausarg-pacbio-hifi-414129-da235337
  single_end:
  - md5sum: bdd8b8ba65c217945353138c1ba0d4be
    url: https://data.bioplatforms.com/dataset/bpa-ausarg-pacbio-hifi-414129-da235337/resource/bdd8b8ba65c217945353138c1ba0d4be/download/414129_AusARG_AGRF_DA235337.ccs.bam
- data_type: PACBIO_SMRT
  name: bpa-ausarg-pacbio-hifi-414129-da235386
  single_end:
  - md5sum: 27fc8f6c8fb9e3b462a3cdae002bd5f4
    url: https://data.bioplatforms.com/dataset/bpa-ausarg-pacbio-hifi-414129-da235386/resource/27fc8f6c8fb9e3b462a3cdae002bd5f4/download/414129_AusARG_AGRF_DA235386.ccs.bam
- data_type: Hi-C
  name: bpa-ausarg-hi-c-414130-hkwjjdmxy
  r1:
  - md5sum: 796d9cd507b33eb34b3b31e4200129c7
    url: https://data.bioplatforms.com/dataset/bpa-ausarg-hi-c-414130-hkwjjdmxy/resource/796d9cd507b33eb34b3b31e4200129c7/download/414130_AusARG_BRF_HKWJJDMXY_AAGCATCG_S5_R1_001.fastq.gz
  r2:
  - md5sum: 71e2e0c5384d238811bd78e54cbe0111
    url: https://data.bioplatforms.com/dataset/bpa-ausarg-hi-c-414130-hkwjjdmxy/resource/71e2e0c5384d238811bd78e54cbe0111/download/414130_AusARG_BRF_HKWJJDMXY_AAGCATCG_S5_R2_001.fastq.gz
scientific_name: Heteronotia binoei
taxon_id: 13085

```

</details>

## Overview

The assembly process has three main steps:

1. Assembly with
   [sanger-tol/genomeassembly](https://github.com/sanger-tol/genomeassembly/)
2. Decontamination with [sanger-tol/ascc](https://github.com/sanger-tol/ascc/)
3. Preparation of curation materials with
   [sanger-tol/treeval](https://github.com/sanger-tol/treeval/), if there are
   Hi-C reads.

The config files for these steps are in the [config](./config) directory.

The sanger-tol workflows are plumbed together by the [included Snakemake
worfklow](./workflow/Snakefile).

## Running the assembly

### Setting up

Clone this repo to the HPC where it will be run.

A profile will be needed to configure the job scheduler on the HPC. Profiles
for [Setonix](./profiles/pawsey) and [Spartan (partial)](./profiles/spartan)
are included. A profile for [local testing](./profiles/local) is also included.

<details>

<summary>More information about the profile</summary>

#### The profile needs at least the following files:

- Snakemake [**job config**](./profiles/pawsey/config.v9+.yaml) and [**workflow
  config**](./profiles/pawsey/workflow.config.yaml): configure the jobs from the
  genome-launcher-workflow.
- [**nextflow config**](./profiles/pawsey/pawsey.config): configure the
  processes from the Sanger-Tol pipelines
- [**ascc.params.config**](./profiles/pawsey/ascc.params.config): the [YAML
  params
  file](https://pipelines.tol.sanger.ac.uk/ascc/0.6.0/usage#running-the-pipeline)
  for ASCC (not shared with the other pipelines).

</details>

### Steps

1. Download the reads and run the QC scripts using the genome-launcher-workflow
   target `pre_genomeassembly`.
2. Run the `genomeassembly` workflow. See the [example 20_genomeassembly.sh
   script](profiles/pawsey/20_genomeassembly.sh).
3. Stage the ASCC reference data using the genome-launcher-workflow target
   `post_genomeassembly`.
4. Run the `ascc` workflow. See the [example 30_ascc.sh
   script](profiles/pawsey/30_ascc.sh).
5. Convert the ASCC output for TreeVal using the genome-launcher-workflow
   target `post_ascc`.
6. If Hi-C is available, run the `treeval` workflow. See the [example
   40_treeval.sh submission script](profiles/pawsey/40_treeval.sh).
7. Run the `post_treeval` target to upload the results to object storage. The
   `post_*` targets all upload the output of the preceding pipeline to object
   storage.


<details>

<summary>Worked example</summary>

#### Run this assembly on Setonix:

> [!IMPORTANT]
>
> The pull command requires a Personal Access Token with read access to code
> and metadata.

1. Pull the repo:
   1. `git init . `
   2. `git remote add origin https://github.com/AToL-Bioinformatics/rHetBin1.2.git`
   3. `git pull origin main`
2. Set up the directory structure: `bash profiles/pawsey/00_preflight.sh`
3. Run the workflow steps:
   1. `sbatch profiles/pawsey/10_pre_genomeassembly.sh`
   2. `sbatch profiles/pawsey/20_genomeassembly.sh`
   3. `sbatch profiles/pawsey/25_post_genomeassembly.sh`
   4. `sbatch profiles/pawsey/30_ascc.sh`
   5. `sbatch profiles/pawsey/35_post_ascc.sh`
   6. `sbatch profiles/pawsey/40_treeval.sh`
   7. `sbatch profiles/pawsey/45_post_treeval.sh`

</details>