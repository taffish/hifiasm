# taf-hifiasm

TAFFISH wrapper for [hifiasm](https://github.com/chhylp123/hifiasm), a fast
haplotype-resolved de novo assembler for PacBio HiFi reads, ONT R10 reads,
Hi-C phasing, trio binning, ultralong-read integration, and T2T-oriented
assembly modes.

This app packages upstream hifiasm `0.25.0` as a single-command TAFFISH tool.
The upstream binary reports the internal source version `0.25.0-r726` via
`hifiasm --version`.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install hifiasm
```

Install the exact release:

```sh
taf install hifiasm 0.25.0-r1
```

For local testing before the app is published to the public index:

```sh
taf install --from .
```

## Usage

Show TAFFISH app help:

```sh
taf-hifiasm --help
```

Show the TAFFISH package version:

```sh
taf-hifiasm --version
```

Show upstream hifiasm help:

```sh
taf-hifiasm -- -h
taf-hifiasm hifiasm -h
```

Show upstream internal source version:

```sh
taf-hifiasm -- --version
taf-hifiasm hifiasm --version
```

Assemble PacBio HiFi reads:

```sh
taf-hifiasm -o sample.asm -t 32 sample.hifi.fq.gz 2> sample.asm.log
```

For very small test datasets, disable the initial bloom filter:

```sh
taf-hifiasm -o tiny -t 4 -f0 tiny.fq.gz 2> tiny.log
```

Assemble an inbred or mostly homozygous sample without duplication purging:

```sh
taf-hifiasm -o sample.asm -t 32 -l0 sample.hifi.fq.gz 2> sample.asm.log
```

Assemble ONT R10 simplex reads:

```sh
taf-hifiasm -o ont.asm --ont -t 64 ont.reads.fq.gz 2> ont.asm.log
```

Use paired-end Hi-C reads for phasing:

```sh
taf-hifiasm -o sample.asm -t 32 --h1 hic_R1.fq.gz --h2 hic_R2.fq.gz sample.hifi.fq.gz 2> sample.asm.log
```

Use parental k-mer databases for trio binning:

```sh
taf-hifiasm -o sample.asm -t 32 -1 paternal.yak -2 maternal.yak sample.hifi.fq.gz 2> sample.asm.log
```

Use ultralong ONT reads in a hybrid assembly:

```sh
taf-hifiasm -o sample.asm -t 32 --ul ultralong.fq.gz sample.hifi.fq.gz 2> sample.asm.log
```

Use self-scaffolding and a human telomere motif for T2T-style assembly:

```sh
taf-hifiasm -o sample.asm -t 32 --dual-scaf --telo-m CCCTAA sample.hifi.fq.gz 2> sample.asm.log
```

Because this is a command-mode TAFFISH tool, the first non-option argument can
also be an executable inside the same container:

```sh
taf-hifiasm hifiasm -h
```

For the main hifiasm command, the shorter option-leading form is usually the
most convenient:

```sh
taf-hifiasm -o sample.asm -t 32 reads.fq.gz
```

## Outputs

hifiasm writes files under the prefix given by `-o`. Common outputs include:

```text
<prefix>.bp.p_ctg.gfa        primary contig assembly graph
<prefix>.bp.hap1.p_ctg.gfa   partially phased haplotype 1 contigs
<prefix>.bp.hap2.p_ctg.gfa   partially phased haplotype 2 contigs
<prefix>.dip.hap1.p_ctg.gfa  trio-binning haplotype 1 contigs, when -1/-2 are used
<prefix>.dip.hap2.p_ctg.gfa  trio-binning haplotype 2 contigs, when -1/-2 are used
<prefix>.ec.bin              corrected-read cache
<prefix>.ovlp.source.bin     overlap cache
<prefix>.ovlp.reverse.bin    reverse-overlap cache
```

hifiasm reuses overlap/cache files on repeated runs with the same prefix. Use
`-i` when you need to ignore saved overlaps and recompute from raw reads.

The primary assembly is GFA. To convert GFA segments to FASTA, use a graph
utility such as `taf-gfatools`:

```sh
taf-gfatools gfatools gfa2fa sample.asm.bp.p_ctg.gfa > sample.asm.p_ctg.fa
```

The upstream README also shows an `awk` one-liner for extracting `S` records;
that conversion is a shell text-processing step rather than a hifiasm runtime
dependency.

## Package

```text
name: hifiasm
command: taf-hifiasm
version: 0.25.0-r1
kind: tool
image: ghcr.io/taffish/hifiasm:0.25.0-r1
```

## Container

The container image is built from `docker/Dockerfile`. It starts from
`debian:12-slim`, downloads the upstream `0.25.0` source archive, verifies its
checksum, applies the same arm64 support patch used by the Bioconda recipe,
compiles hifiasm from source, and copies only the runtime binary, man page, and
small upstream documentation into the final image.

Included runtime components:

```text
hifiasm
zlib runtime library
upstream README, LICENSE, and hifiasm.1 man page
```

The image is built and validated for:

```text
linux/amd64
linux/arm64
```

No Python, R, Java, GPU runtime, database, or model download is required for
the hifiasm binary itself.

## Boundaries

The image intentionally does not bundle `yak`. Trio-binning mode uses
precomputed parental `.yak` files, but hifiasm does not create those databases
internally. Generate them before running hifiasm, for example with a dedicated
yak app or another trusted yak installation.

hifiasm also does not replace downstream graph conversion, assembly evaluation,
polishing, scaffolding, or visualization tools. Compose those steps with other
TAFFISH apps or workflow code as needed.

The public smoke test checks the binary, version/help surface, `zlib` linkage,
and a tiny offline run with `-f0`. It does not validate biological assembly
quality on real HiFi, ONT, Hi-C, ultralong, or trio datasets.

## Upstream

- Source: <https://github.com/chhylp123/hifiasm>
- Documentation: <https://hifiasm.readthedocs.io/en/latest/>
- Release: <https://github.com/chhylp123/hifiasm/releases/tag/0.25.0>
- License: MIT
- Citation: Cheng et al. 2021; Cheng et al. 2022; Cheng et al. 2024
- DOI: `10.1038/s41592-020-01056-5`
- DOI: `10.1038/s41587-022-01261-x`
- DOI: `10.1038/s41592-024-02269-8`
