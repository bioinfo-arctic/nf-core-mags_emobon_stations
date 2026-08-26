# EMOBON nf-core/mag workflows

This repository documents the processing and analysis of EMOBON metagenomic samples using **nf-core/mag**. It serves as a reproducible record of the workflow, including pipeline configurations, commands, computational decisions, troubleshooting, and downstream analyses of metagenome-assembled genomes (MAGs).

The repository is intended both as a working record of the analyses and as a resource for collaborators who need to understand, reproduce, or extend the EMOBON MAG workflow.

Documentation includes:

* nf-core/mag commands and configuration files
* sample- and station-specific processing strategies
* computational resource optimization and troubleshooting
* assembly and genome-binning decisions
* BUSCO-based MAG assessment and identification of eukaryotic MAGs
* taxonomic assignment and validation approaches
* downstream analyses of recovered MAGs
* notes explaining deviations from default nf-core/mag parameters
* lessons learned while processing large EMOBON metagenomic datasets

The repository will evolve alongside the EMOBON MAG analyses and provide a traceable record of methodological decisions and workflow development.

## Running nf-core/mag

The EMOBON metagenomic datasets are processed using **nf-core/mag v5.3.0**. Multiple stations can be submitted in the same Nextflow execution, with each station represented as an independent sample in the nf-core/mag samplesheet.

For example:

```text
sample,group,short_reads_platform,short_reads_1,short_reads_2,long_reads
STATION_A,0,ILLUMINA,/path/to/STATION_A_R1.fastq.gz,/path/to/STATION_A_R2.fastq.gz,
STATION_B,1,ILLUMINA,/path/to/STATION_B_R1.fastq.gz,/path/to/STATION_B_R2.fastq.gz,
```

The workflow can then be executed as:

```bash
nextflow run nf-core/mag \
    -r 5.3.0 \
    -profile docker \
    -c configs/spygene2_nfcore_mag.config \
    --input samplesheet.csv \
    --outdir results/ \
    --skip_spades \
    --skip_concoct \
    --skip_semibin \
    --skip_maxbin2 \
    --skip_gtdbtk \
    --megahit_options "--k-list 29,39,59,79,99,119,141" \
    -resume
```

### Important modifications from the default workflow

The configuration has been optimized for very large EMOBON Illumina metagenomic datasets and for the computational resources available on `spygene2`.

#### MEGAHIT k-mer range

MEGAHIT is run with:

```text
--k-list 29,39,59,79,99,119,141
```

rather than starting at the default smaller k-mer size.

During processing of large EMOBON datasets, the k=21 assembly stage generated an exceptionally large de Bruijn graph and resulted in excessive memory consumption and out-of-memory termination. Starting at k=29 substantially reduced this initial memory requirement while retaining the subsequent k-mer sizes used for assembly.

MEGAHIT is therefore allocated up to **32 CPUs and 1.2 TB RAM** for these datasets.

#### Bowtie2 assembly index

The Bowtie2 assembly-index construction step is configured with:

```text
cpus   = 1
memory = 256.GB
time   = 96.h
```

Large metagenomic assemblies can require substantial memory during index construction. Additional CPUs provide little benefit for the implementation used by this workflow step.

#### Bowtie2 read-to-assembly alignment

The read-to-assembly mapping step:

```text
NFCORE_MAG:MAG:BINNING_PREPARATION:
SHORTREAD_BINNING_PREPARATION:
BOWTIE2_ASSEMBLY_ALIGN
```

is configured with:

```text
cpus   = 16
memory = 512.GB
time   = 96.h
```

The CPU allocation was increased to **16 cores** because mapping the original reads back to large assemblies represents a substantial computational bottleneck for the EMOBON datasets.

This mapping is required for estimating contig coverage used during downstream genome binning.

### Portability

The supplied configuration records the resources and parameters used for the EMOBON analyses on `spygene2`. The scientific parameters can be reused on other systems, but CPU, memory, execution-time, container, and filesystem settings should be adapted to the available computing environment.

In particular, the large memory allocations in this configuration should be interpreted as upper resource limits established from experience with the largest EMOBON datasets rather than minimum hardware requirements for nf-core/mag.

