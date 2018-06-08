# DegePrime

## Content
1. Overview
2. Usage

### 1. Overview

DegePrime is a program that for each position in a multiple sequence alignment finds a degenerate oligomer, of defined length and degeneracy, of as high coverage as possible (matching as many of the sequences as possible in that position). It hence attempts to solve the "maximum coverage degenerate primer design problem" (MC-DPD), and uses a novel heuristic for this. The main script is DegePrime.pl that performs the actual oligomer selection procedure. The script TrimAlignment.pl is used to prepare the alignment file and has to be run prior to DegePrime.pl. As an option, DegePrime.pl can output primer coverage among taxonomic groups of sequences, if a file with taxonomic information for the sequences is provided. Such a file can be generated from an RDP (http://rdp.cme.msu.edu/) file in genbank format by the script MakeRdpTaxonomy.pl or from a Silva (http://www.arb-silva.de/) file in fasta format with taxonomies in the sequence headers by the script MakeSilvaTaxonomy.pl.

### 2. Usage

Input files

1) As input, an alignment file in fasta format is required. Gaps should be indicated by "-". Gaps in the start and end of a sequence can be indicated by "." or "_". All sequences must be of the same length (including gaps).


Running

"Spare me the details...":
```
 perl TrimAlignment.pl -i align_file -min cutoff -o trimmed_align_file
```
 (-min 0.9 usually works well)
```
 perl DegePrime.pl -i trimmed_align_file -d degeneracy -l primer_length -o output_file
```
 (e.g. -d 12 -l 18)


"Tell me more!":

First, the alignment file has to be prepared by the script TrimAlignment.pl. There are two reasons for this. First, your alignment may include many positions with gaps in many of the sequences, especially if the alignment is based on a large number of sequences. Since DegePrime.pl will, in each window, only use those sequences that do not have gaps, this will be troublesome. With TrimAlignment.pl you can remove those columns in your alignment that are not occupied (have a nucleotide) in at least a defined proportion of sequences, set by the optional parameter -min :

```
 perl TrimAlignment.pl -i align_file -min cutoff -o trimmed_align_file
```

where (0 =< cutoff =< 1). Using -min 1 will hence only output columns that have a nucleotide in every sequence. If you don't specify -min the value 0 is used, so:
```
 perl TrimAlignment.pl -i align_file -o trimmed_align_file
```
will output the same alignment as the original. There is however one important difference - any lower case letter in the original alignment will be converted to upper case. This is important because DegePrime.pl interprets lower case and upper case letters differently! Lower case letters will be regarded as preceding a gap in the original alignment that has been removed in the trimmed alignment, while upper case letters will not. DegePrime.pl uses this information when it designs degenerate primers and calculates coverages (it will only use sequences without gaps in the window when searching for optimal degenerate primers, and when calculating the primer coverage it will consider sequences with gaps in the window as not matching the primer). Therefor, if your original alignment has lower case letters, TrimAlignment will convert all these to upper case, and, if it removes any gaps, nucleotides preceding these will be changed to lower case. You should thus run TrimAlignment.pl even though you don't want to remove any columns (unless you are sure your alignment only includes upper case letters).


Now we are ready for finding degenerate primers (using the output file from TrimAlignment.pl):
```
 perl DegePrime.pl -i trimmed_align_file -d degeneracy -l primer_length -o output_file
```
-d should be a possible degeneracy, i.e. 1, 2, 3, 4, 6, 8, 9, 12, and so forth (or more generally a number > 0 that can be expressed as 2^i * 3^j, where i and j are integers or 0).


The output_file will be a tab-separated text file that includes the following columns:

Pos	TotalSeq	UniqueMers	Entropy	PrimerDeg	PrimerMatching	PrimerSeq

Pos: 			Position of the (first base in the) window in the trimmed_align_file (starting at 0).
TotalSeq:		Number of sequences that span this window (including sequences with internal gaps in this region).
UniqueMers:		Number of unique oligomer sequences (without in/dels) in the window.
Entropy: 		Entropy of the window, calculated as -Σ Pi log2(Pi), where Pi is the frequency of oligomer i.  
PrimerDeg:		Degeneracy of the selected primer.
PrimerMatching:		Number of sequences that match the selected primer.
PrimerSeq:		Sequence of the selected degenerate primer.


For more help and additional optional parameters for the scripts, use -h: 
```
 perl DegePrime.pl -h

 perl TrimAlignment.pl -h

 perl MakeRdpTaxonomy.pl -h

 perl MakeSilvaTaxonomy.pl -h
 ```
