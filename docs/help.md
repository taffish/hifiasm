taf-hifiasm 0.25.0-r1

TAFFISH wrapper for hifiasm, a fast haplotype-resolved de novo assembler for
PacBio HiFi reads, ONT R10 reads, Hi-C phasing, trio binning, ultralong-read
integration, and T2T-oriented assembly modes.

Usage:
  taf-hifiasm [TAF-APP-OPTION]
  taf-hifiasm [HIFIASM-OPTION]... READS...
  taf-hifiasm [IN-CONTAINER-COMMAND] [ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Recommended commands:
  taf-hifiasm --help
  taf-hifiasm --version
  taf-hifiasm -- -h
  taf-hifiasm -- --version
  taf-hifiasm hifiasm -h
  taf-hifiasm hifiasm --version
  taf-hifiasm -o sample.asm -t 32 sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o tiny -t 4 -f0 tiny.fq.gz 2> tiny.log
  taf-hifiasm -o sample.asm -t 32 -l0 sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o ont.asm --ont -t 64 ont.reads.fq.gz 2> ont.asm.log
  taf-hifiasm -o sample.asm -t 32 --h1 hic_R1.fq.gz --h2 hic_R2.fq.gz sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o sample.asm -t 32 -1 paternal.yak -2 maternal.yak sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o sample.asm -t 32 --ul ultralong.fq.gz sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o sample.asm -t 32 --dual-scaf --telo-m CCCTAA sample.hifi.fq.gz 2> sample.asm.log

Main input modes:
  HiFi reads             default hifiasm mode for PacBio HiFi reads
  ONT R10 reads          use --ont with ONT simplex R10 FASTQ input
  Hi-C phasing           use --h1 and --h2 with paired-end Hi-C FASTQ files
  Trio binning           use -1 and -2 with precomputed parental .yak files
  Ultralong integration  use --ul with ultralong ONT reads
  T2T-style assembly     combine --dual-scaf, --telo-m, Hi-C, and/or --ul as appropriate

Common options:
  -o STR       Output prefix
  -t INT       Number of worker threads
  -f0          Disable the initial bloom filter; useful for tiny test datasets
  -l0          Disable duplication purging; useful for inbred/homozygous samples
  -i           Ignore saved overlaps and recompute from raw reads
  --ont        Assemble ONT R10 simplex reads instead of PacBio HiFi reads
  --h1 FILE    Hi-C read 1 FASTQ for phasing
  --h2 FILE    Hi-C read 2 FASTQ for phasing
  -1 FILE      Paternal yak k-mer database for trio binning
  -2 FILE      Maternal yak k-mer database for trio binning
  --ul FILE    Ultralong ONT reads for hybrid/T2T assembly
  --dual-scaf  Use self-scaffolding between haplotypes
  --telo-m STR Telomere motif, for example CCCTAA for human assemblies

Output files:
  <prefix>.bp.p_ctg.gfa        Primary contig assembly graph
  <prefix>.bp.hap1.p_ctg.gfa   Partially phased haplotype 1 graph
  <prefix>.bp.hap2.p_ctg.gfa   Partially phased haplotype 2 graph
  <prefix>.dip.hap1.p_ctg.gfa  Trio-binning haplotype 1 graph, when -1/-2 are used
  <prefix>.dip.hap2.p_ctg.gfa  Trio-binning haplotype 2 graph, when -1/-2 are used
  <prefix>.ec.bin              Corrected-read cache
  <prefix>.ovlp.source.bin     Overlap cache
  <prefix>.ovlp.reverse.bin    Reverse-overlap cache

Notes:
  - This command runs hifiasm inside the TAFFISH container image.
  - taf-hifiasm --help and taf-hifiasm --version are handled by the TAFFISH
    wrapper. Use taf-hifiasm -- -h or taf-hifiasm hifiasm -h for upstream help.
  - hifiasm is a single upstream executable, so option-leading commands such as
    taf-hifiasm -o sample.asm -t 32 reads.fq.gz are the normal concise form.
  - The packaged upstream release is hifiasm 0.25.0. The binary's own
    --version output is the internal source identifier 0.25.0-r726.
  - hifiasm reuses <prefix>.*.bin overlap/cache files. Use -i to force
    recomputation from raw reads.
  - Trio-binning mode consumes .yak files but does not create them. Generate
    parental yak databases before running hifiasm.
  - GFA-to-FASTA conversion is a downstream graph/text-processing step. Use
    taf-gfatools or another trusted utility when you need FASTA output.
  - The image includes hifiasm and its zlib runtime only. It does not bundle
    yak, graph converters, assembly evaluation tools, polishers, scaffolders,
    or visualization tools.
  - Full genome assemblies can need many CPU threads, large memory, and hours
    to days of runtime. The TAFFISH smoke test only checks a tiny offline run.

Container:
  image: ghcr.io/taffish/hifiasm:0.25.0-r1
  supported backends: apptainer, podman, docker
  supported platforms: linux/amd64, linux/arm64

Upstream:
  project: hifiasm
  source:  https://github.com/chhylp123/hifiasm
  docs:    https://hifiasm.readthedocs.io/en/latest/
  release: https://github.com/chhylp123/hifiasm/releases/tag/0.25.0
  license: MIT
  citation: Cheng et al. 2021; Cheng et al. 2022; Cheng et al. 2024
  doi: 10.1038/s41592-020-01056-5
  doi: 10.1038/s41587-022-01261-x
  doi: 10.1038/s41592-024-02269-8
