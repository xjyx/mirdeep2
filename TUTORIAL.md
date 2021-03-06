# miRdeep2 tutorial

To run the tutorial please go to the tutorial subfolder.


## Introduction:

The user wishes to analyze deep sequencing data mapping to a ~6 kb region on
*C. elegans* chromosome II for known and novel miRNA genes.

---


## Preliminary files

| file                             | description
|----------------------------------|------------------------------------------|
| `cel_cluster.fa`                 | a FASTA file with the reference genome (this file is in fact a ~6 kb region of the *C. elegans* chromosome II). |
| `mature_ref_this_species.fa`     | a FASTA file with the reference miRBase mature miRNAs for the species (*C. elegans* miRBase v.14 mature miRNAs) |
| `mature_ref_other_species.fa`    | a FASTA file with the reference miRBase mature miRNAs for related species (*C. briggsae* and *D. melanogaster* miRBase v.14 mature miRNAs) |
| `precursors_ref_this_species.fa` | a FASTA file with the reference miRBase precursor miRNAs for the species (*C. elegans* miRBase v.14 precursor miRNAs) |
| `reads.fa`                       | a FASTA file with the deep sequencing reads. |


---


## Analysis

### Step 1:

build an index of the genome (in this case the ~6 kb region):

```sh
bowtie-build cel_cluster.fa cel_cluster
```

### Step 2

process reads and map them to the genome.

The `-c` option designates that the input file is a FASTA file (for other input
formats, see the `README.md` file).  The `-j` options removes entries with
non-canonical letters (letters other than `a`, `c`, `g`, `t`, `u`, `n`, `A`,
`C`, `G`, `T`, `U`, or `N`). The `-k` option clips adapters.  The `-l` option
discards reads shorter than 18 nts.  The `-m` option collapses the reads.  The
`-p` option maps the processed reads against the previously indexed genome
(`cel_cluster`).  The `-s` option designates the name of the output file of
processed reads and the `-t` option designates the name of the output file of
the genome mappings. Last, `-v` gives verbose output to the screen.

```sh
mapper.pl reads.fa -c -j -k TCGTATGCCGTCTTCTGCTTGT  -l 18 -m -p cel_cluster \
  -s reads_collapsed.fa -t reads_collapsed_vs_genome.arf -v
```

### Step 3

fast quantitation of reads mapping to known miRBase precursors.

(This step is not required for identification of known and novel miRNAs in the
deep sequencing data when using `miRDeep2.pl`.)

```sh
quantifier.pl -p precursors_ref_this_species.fa -m mature_ref_this_species.fa \
  -r reads_collapsed.fa -t cel -y 16_19
```

The `miRNA_expressed.csv` gives the read counts of the reference miRNAs in the
data in tabular format. The results can also be browsed by opening
`expression_16_19.html` with an internet browser.

### Step 4

identification of known and novel miRNAs in the deep sequencing data:

```sh
miRDeep2.pl reads_collapsed.fa cel_cluster.fa reads_collapsed_vs_genome.arf \
  mature_ref_this_species.fa mature_ref_other_species.fa \
  precursors_ref_this_species.fa -t C.elegans 2> report.log
```

### Step 5

browse the results.

open the `results.html` using an internet browser. Notice that `cel-miR-37` is
predicted twice, since both potential precursors excised from this locus can
fold into hairpins. However, the annotated hairpin scores much higher than the
non-annotated one (miRDeep2 score `6.1e+4` vs. `-0.2`).

---
