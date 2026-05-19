taf-hifiasm 0.25.0-r2

TAFFISH wrapper for hifiasm, a fast haplotype-resolved de novo assembler for
PacBio HiFi, ONT R10, Hi-C phasing, trio binning, ultralong-read integration,
and T2T-oriented assembly modes.

Usage:
  taf-hifiasm [TAF-APP-OPTION]
  taf-hifiasm [HIFIASM-OPTION]... READS...
  taf-hifiasm [IN-CONTAINER-COMMAND] [ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Help and version:
  taf-hifiasm -- -h
  taf-hifiasm -- --version
  taf-hifiasm hifiasm -h
  taf-hifiasm hifiasm --version

Examples:
  taf-hifiasm -o sample.asm -t 32 sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o tiny -t 4 -f0 tiny.fq.gz 2> tiny.log
  taf-hifiasm -o sample.asm -t 32 -l0 sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o ont.asm --ont -t 64 ont.reads.fq.gz 2> ont.asm.log
  taf-hifiasm -o sample.asm -t 32 --h1 hic_R1.fq.gz --h2 hic_R2.fq.gz sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o sample.asm -t 32 -1 paternal.yak -2 maternal.yak sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o sample.asm -t 32 --ul ultralong.fq.gz sample.hifi.fq.gz 2> sample.asm.log
  taf-hifiasm -o sample.asm -t 32 --dual-scaf --telo-m CCCTAA sample.hifi.fq.gz 2> sample.asm.log

Main input modes:
  HiFi reads, ONT R10 reads with --ont, Hi-C phasing with --h1 and --h2, trio
  binning with -1 and -2 parental yak databases, ultralong integration with
  --ul, and T2T-style self-scaffolding with --dual-scaf and --telo-m.

Common options:
  -o STR       Output prefix
  -t INT       Number of worker threads
  -f0          Disable initial bloom filter; useful for tiny tests
  -l0          Disable duplication purging
  -i           Ignore saved overlaps and recompute
  --ont        Assemble ONT R10 simplex reads
  --h1 FILE    Hi-C read 1 FASTQ
  --h2 FILE    Hi-C read 2 FASTQ
  -1 FILE      Paternal yak k-mer database
  -2 FILE      Maternal yak k-mer database
  --ul FILE    Ultralong ONT reads
  --dual-scaf  Use self-scaffolding between haplotypes
  --telo-m STR Telomere motif

Outputs:
  <prefix>.bp.p_ctg.gfa, <prefix>.bp.hap1.p_ctg.gfa,
  <prefix>.bp.hap2.p_ctg.gfa, trio-binning diploid haplotype GFA files, and
  overlap/cache .bin files.

Notes:
  Option-leading commands are the normal concise form for this single binary.
  The package version is 0.25.0; upstream --version reports 0.25.0-r726.
  hifiasm reuses <prefix>.*.bin caches; use -i to recompute from raw reads.
  Trio mode consumes .yak files but does not create them. Use taf-gfatools or
  another trusted tool for GFA-to-FASTA conversion.
  This image does not bundle yak, graph converters, evaluation tools,
  polishers, scaffolders, or visualization tools.

Container:
  image: ghcr.io/taffish/hifiasm:0.25.0-r2
  platforms: linux/amd64, linux/arm64

Upstream:
  source:  https://github.com/chhylp123/hifiasm
  docs:    https://hifiasm.readthedocs.io/
  release: 0.25.0
  license: MIT
  citation: Cheng et al. 2021; Cheng et al. 2022; Cheng et al. 2024
