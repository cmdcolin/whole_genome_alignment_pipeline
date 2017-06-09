# whole_genome_alignment_pipeline


Some guidelines for whole genome alignments

## Alignment commands


### LAST

    lastdb mydb target.fa
    lastal mydb query.fa -f maf > output.maf

### LASTZ

    lastz "target.fa[multiple]" "query.fa[multiple]" --format=maf > output.maf

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

Simplifies http://genomewiki.ucsc.edu/index.php/Whole_genome_alignment_howto#Example.2C_step_4:_Maffing


