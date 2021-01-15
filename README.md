## tmnt: Transposon methylation nanopore tools

Tools for parsing and plotting methylation data from Oxford Nanopore Technologies data in a way that is useful for transposable element applications.

## Installation

`git clone https://github.com/adamewing/tmnt.git` or download a .zip file from GitHub.

install tmnt and dependencies via:

`python setup.py install`

## Input data

### Alignments (.bam)

Alignments stored in .bam format should be sorted and indexed and should use the same read names as the associated methylation data.

### Methylation Calls

TMNT hs functions for loading methylation data can from nanopolish, megalodon, or basecalled guppy fast5s, using the appropriate function below.

Once coverted, the sqlite .db file can be input to TMNT functions (e.g. `segplot`, `locus`, etc).

## Commands:

### db-nanopolish

Load nanopolish methylation into sqlite db. Needs to be done before running methylation parsing or plotting commands.

### db-megalodon
(work in progress)

### db-guppy
(work in progress)

### segmeth

Outputs aggregate methylation / demethylation call counts over intervals. Required before generating strip / violin plots with `segplot`

Requires whitespace-delimited list of segments in a BED-like format: chromosome, start, end, label, strand.

Sample input file `-d/--data` has the following whitespace-delimited fields (one sample per line): BAM file, Methylation DB (generated with e.g. `db-nanopolish`)

Highly recommend parallelising with `-p/--proc` option if possible.

Can be used to generate genome-wide methylation stats aggregated over windows via `bedtools makewindows`.

Example:

Aggregate methylation calls over full length L1 elements (RepeatMasker, hg38), exclude reads which map entirely within L1s:

`tmnt segmeth -d MCF7_data.txt -i L1_FL.bed -p 32 --excl_ambig`

Contents of `MCF7_data.txt`:

```
MCF7_ATCC.haplotag.bam MCF7_ATCC.nanopolish.db
MCF7_Euro.haplotag.bam MCF7_Euro.nanopolish.db
```

Contents of L1_FL.bed (first 10 lines):

```
chr1    440936  447357  L1PA7   +
chr1    675912  682333  L1PA7   +
chr1    7411758 7417922 L1PA4   -
chr1    14355445        14361840        L1PA7   -
chr1    14382319        14388341        L1PA3   -
chr1    25506586        25512707        L1PA2   +
chr1    30427133        30433546        L1MA4   +
chr1    30453586        30459718        L1PA5   -
chr1    30493390        30499556        L1PA4   +
chr1    33671998        33678118        L1PA4   +
```

Output (`)L1_FL.MCF7_data.excl_ambig.segmeth.tsv`), first 10 lines:

```
seg_id  seg_chrom       seg_start       seg_end seg_name        seg_strand      MCF7_ATCC.haplotag_meth_calls   MCF7_ATCC.haplotag_unmeth_calls MCF7_ATCC.haplotag_no_calls     MCF7_ATCC.haplotag_methfrac     MCF7_Euro.haplotag_meth_calls     MCF7_Euro.haplotag_unmeth_calls MCF7_Euro.haplotag_no_calls     MCF7_Euro.haplotag_methfrac
chr1:440936-447357      chr1    440936  447357  L1PA7   +       0       0       0       NaN     0       0       0       NaN
chr1:675912-682333      chr1    675912  682333  L1PA7   +       129     66      141     0.6615384615384615      222     55      133     0.8014440433212996
chr1:7411758-7417922    chr1    7411758 7417922 L1PA4   -       491     410     867     0.5449500554938956      349     79      364     0.8154205607476636
chr1:14355445-14361840  chr1    14355445        14361840        L1PA7   -       222     302     455     0.42366412213740456     293     168     406     0.6355748373101953
chr1:14382319-14388341  chr1    14382319        14388341        L1PA3   -       484     364     557     0.5707547169811321      469     149     563     0.7588996763754046
chr1:25506586-25512707  chr1    25506586        25512707        L1PA2   +       727     495     554     0.5949263502454992      239     276     205     0.4640776699029126
chr1:30427133-30433546  chr1    30427133        30433546        L1MA4   +       35      48      78      0.42168674698795183     20      27      48      0.425531914893617
chr1:30453586-30459718  chr1    30453586        30459718        L1PA5   -       309     297     516     0.5099009900990099      189     42      200     0.8181818181818182
chr1:30493390-30499556  chr1    30493390        30499556        L1PA4   +       460     381     673     0.5469678953626635      328     86      505     0.7922705314009661
```


### segplot

Generates strip plots or violin plots (`-v/--violin`) from `segmeth` output.

### locus

Generates smoothed methylation profiles across specific loci with many configurable parameters for one or more samples.

Sample input file `-d/--data` has the following whitespace-delimited fields (one sample per line): BAM file, Methylation DB (generated with e.g. `db-nanopolish`)

### haplocus

Generates smoothed haplotype-aware methylation profiles across specific loci for one sample. Requires haplotypes to be tagged with whatshap.

### composite

Generates "composite" methylation plots where multiple per-element profiles are aligned to and plotted against a reference sequence.

### wgmeth

Outputs genome-wide statistics on CpGs covered by at least one call. By default, `wgmeth` yields output in bedMethyl format (0-based coordinates).

The `--dss` option yields genome-wide files suitable for input to DSS for statistical assessment of differential methylation (1-based coordinates).
