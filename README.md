#CIRCexplorer

[![Build Status](https://travis-ci.org/YangLab/CIRCexplorer.svg?branch=master)](https://travis-ci.org/YangLab/CIRCexplorer)

CIRCexplorer is a combined strategy to identify junction reads from back spliced exons and intron lariats.

Version: 1.0.2

Authors: Xiao-Ou Zhang (zhangxiaoou@picb.ac.cn), Li Yang (liyang@picb.ac.cn)

[Download the latest stable version of CIRCexplorer](http://github.com/Yanglab/CIRCexplorer/releases)

To see what has changed in recent versions of CIRCexplorer, see the [CHANGELOG](https://github.com/YangLab/CIRCexplorer/blob/master/CHANGELOG.md).

##A schematic flow shows the pipeline

![pipeline](https://raw.githubusercontent.com/YangLab/CIRCexplorer/master/flow.jpg)

##Prerequisites

###Software / Package

* [TopHat](http://ccb.jhu.edu/software/tophat/index.shtml) 2.0.9
* [TopHat-Fusion](http://ccb.jhu.edu/software/tophat/fusion_index.html) included in TopHat 2.0.9
* [bedtools](https://github.com/arq5x/bedtools2)
* [SAMtools](http://samtools.sourceforge.net)
* [pysam](https://github.com/pysam-developers/pysam)
* [interval](https://github.com/kepbod/interval)
* [docopt](https://github.com/docopt/docopt)

###RNA-seq

The poly(A)−/ribo− RNA-seq is recommended. If you want to obtain more circular
RNAs, RNase R treatment could be performed. To obtain junction reads for
circular RNAs, two-step mapping strategy was exploited:

* Multiple mapping with TopHat

```bash
tophat2 -a 6 --microexon-search -m 2 -p 10 -G knownGene.gtf -o tophat hg19_bowtie2_index RNA_seq.fastq
```

* Convert unmapped reads (using bamToFastq from [bedtools](https://github.com/arq5x/bedtools2))

```bash
bamToFastq -i tophat/unmapped.bam -fq tophat/unmapped.fastq
```

* Unique mapping with TopHat-Fusion

```bash
tophat2 -o tophat_fusion -p 15 --fusion-search --keep-fasta-order --bowtie1 --no-coverage-search hg19_bowtie1_index tophat/unmapped.fastq
```

##Usage

```bash
CIRCexplorer.py -- circular RNA analysis toolkits.

Usage: CIRCexplorer.py [options]

Options:
    -f FUSION, --fusion=FUSION      Fusion BAM file
    -g GENOME, --genome=GENOME      Genome FASTA file
    -r REF, --ref=REF               Gene annotation
    -o OUT, --output=OUT            Output file [default: circ.txt]
```

###Example

```bash
CIRCexplorer.py -f tophat_fusion/accepted_hits.bam -g hg19.fa -r ref.txt
```

###Note

* ref.txt is in the format (refFlat.txt in UCSC) below (see details in [the example file](https://github.com/YangLab/CIRCexplorer/blob/master/example/ref_example.txt))

| Field       | Description                   |
| :---------: | :---------------------------- |
| geneName    | Name of gene                  |
| isoformName | Name of isoform               |
| chrom       | Reference sequence            |
| strand      | + or - for strand             |
| txStart     | Transcription start position  |
| txEnd       | Transcription end position    |
| cdsStart    | Coding region start           |
| cdsEnd      | Coding region end             |
| exonCount   | Number of exons               |
| exonStarts  | Exon start positions          |
| exonEnds    | Exon end positions            |

* hg19.fa is genome sequence in FASTA format and indexed (using `samtools faidx` from [SAMtools](http://samtools.sourceforge.net))

##Output

See details in [the example file](https://github.com/YangLab/CIRCexplorer/blob/master/example/output_example.txt)

| Field       | Description                           |
| :---------: | :------------------------------------ |
| chrom       | Chromosome                            |
| start       | Start of junction                     |
| end         | End of junction                       |
| name        | Circular RNA/Junction reads           |
| score       | 1 to indicate reads were realigned    |
| strand      | + or - for strand                     |
| thickStart  | No meaning                            |
| thickEnd    | No meaning                            |
| itemRgb     | 0,0,0                                 |
| exonCount   | Number of exons                       |
| exonSizes   | Exon sizes                            |
| exonOffsets | Exon offsets                          |
| readNumber  | Number of junction reads              |
| flag        | 'Yes' for ciRNA, and 'No' for circRNA |
| geneName    | Name of gene                          |
| isoformName | Name of isoform                       |
| flankIntron | Left intron/Right intron              |

***Note: The first 12 columns are in [BED12 format](http://genome.ucsc.edu/FAQ/FAQformat.html#format1).***

##Citation

**[Zhang et al., Complementary Sequence-Mediated Exon Circularization, Cell (2014), http://dx.doi.org/10.1016/j.cell.2014.09.001](http://www.sciencedirect.com/science/article/pii/S0092867414011118)**

##License

Copyright (C) 2014 YangLab.  See the [LICENSE](https://github.com/YangLab/CIRCexplorer/blob/master/LICENSE.txt)
file for license rights and limitations (MIT).
