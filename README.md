# whole_genome_alignment_pipeline


Some guidelines for whole genome alignments


## Masking

If you have a RepeatMasker.gff, you can softmask (replace repeats with lower case in FASTA) with bedtools

    bedtools maskfasta -soft -fi genome.fa -bed repeastmasker.gff -fo genome_softmask.fa
    

## Whole genome alignments

### LAST

    lastdb mydb target.fa
    lastal mydb query.fa -f maf > output.maf


#### Notes

* LAST claims to be able to handle repeats very flexibly and does not even require repeatmasking by some benchmarks
* LAST allows multiple seeding schemes to be used. The lastdb -uMAM8 has 8 seeding patterns and requires high memory, -uMAM4 has 4 and requires less memory for example. -uNEAR is recommended for short-and-exact matches. See http://last.cbrc.jp/doc/last-seeds.html

### LASTZ

    lastz "target.fa[multiple]" "query.fa[multiple]" --format=maf > output.maf

#### Notes

* LASTZ excludes softmasked sequences from the seeding stage of the alignment process but not from later stages
* Seeds can be thought of as an alternative to an exact k-mer match, and is a pattern such as 0100100011 where the "1" are positions that match
* LASTZ allows "anchors" step to skip seeding via the --segments argument. The input is a BLASTtab-like format, but anchors must be "same length"

### MUMMER

    nucmer -p output target.fa query.fa
    delta2maf output.delta > out.maf

## Post-processing

Eventually the netToAxt command requires 2bit format files and many steps require the "chrom.sizes" type file (two columns sequence name and length)

```
samtools faidx query.fa
samtools faidx target.fa
cut -f1,2 query.fa.fai > query.sizes
cut -f1,2 target.fa.fai > target.sizes
faToTwoBit query.fa query.2bit
faToTwoBit target.fa target.2bit
```

### Chaining and netting

As mentioned before, if your whole genome alignment can get into MAF format (or directly output PSL), you can run these steps fairly easily

```
maf-convert psl output.maf > last.psl
axtChain -linearGap=loose -psl last.psl -faQ -faT target.fa query.fa out.pre.chain
chainPreNet out.pre.chain target.sizes query.sizes out.chain
chainNet out.chain target.sizes query.sizes target.net query.net
netToAxt target.net out.chain target.2bit query.2bit out.axt
axtToMaf out.axt target.sizes query.sizes out.maf
```

## Pre-requisites

    brew install blat kent-tools samtools last lastz mummer
    
Works with linuxbrew or OSX

If you can get your whole genome aligner to output MAF, then you can do the post-processing steps. Note that delta2maf from mummer/nucmer is re-hosted in this repository for convenience, it runs on linux systems only. delta2maf is copyright of the original developers for Mugsy/Mummer and destributed unmodified under the Artistic License 2.0

## References

Inspired by http://rstudio-pubs-static.s3.amazonaws.com/183476_e0f3e8b454434690a45aff9dff48fdfb.html#last-aligner

Simplifies all the complicated steps listed here http://genomewiki.ucsc.edu/index.php/Whole_genome_alignment_howto

## Notes

This is a work in progress and experimental


## Benchmarks


### Aligning two draft vertebrate genomes

* Initial output `LAST aligner, -uMAM4`: 767,262 alignments  
* After running `axtChain -linearGap=loose`: 368,085 alignments



