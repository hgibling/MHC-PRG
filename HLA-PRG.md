# HLA-PRG

HLA\*PRG is an algorithm for HLA type inference from next-generation sequencing data.

## Introduction

HLA*PRG infers HLA types by

1. Aligning reads putatively originating from the HLA genes to a Population Reference Graph (PRG) of the HLA genes
2. Evaluating the graph-aligned reads in a simple likelihood framework - for each gene we find the pair of underlying alleles that maximizes the probability of observing the reads aligned to the locus

HLA types are inferred at 6-digit "G" resolution. 6-digit "G" resolution specifies the sequence of the exons encoding the antigen-binding site of the HLA protein - (i.e. exons 2 and 3 for HLA class I genes and exon 2 for HLA class II genes) - all alleles with identical sequences over these exons end up in the same "G" group. 6-digit "G" resolution is a useful metric to consider - most biological variability associated with the HLA genes is thought to come from differences in peptide binding, and this in turn is governed mostly by the amino acid sequence of the peptide binding site.

In our evaluations (still to be published), we find HLA\*PRG to be highly accurate if applied to high-quality (PCR-free, >30x coverage) whole-genome sequencing data. We don't recommend applying HLA*PRG to exome sequencing data.

The inference process always begins with a (GRCh37, see below) BAM file and consists of four steps:

1. Positive selection - extract read pairs that might come from the HLA genes (according to some k-Mer metrics)
2. Negative selection - remove read pairs from the set so-created that are likely to come from other genomic locations (again, using some k-Mer metrics)
3. Read-to-PRG alignment
4. HLA type inference

The rationale for splitting the read selection process into two separate components is that the two steps have different resource footprints - positive selection is typically applied to individual BAM files (e.g. on a cluster) and not very memory-intensive. Negative selection, by contrast, is memory-intensive and typically applied to batches of samples (if applicable; this is only for computational reasons - negative selection is independent-per-sample, statistically speaking).

### GRCh37

Currently HLA\*PRG should only be applied to B37-aligned BAM files. We will soon update the pipeline to deal with GRCh38-based BAMs.

### Computational considerations
Applying HLA\*PRG is currently computationally intensive - we are working on an optimized version of the algorithms.

Analyzing the NA12878 Illumina PLatinum data (2 x 100bp, 55x) takes on the order of 55 CPU hours and about 75G of RAM (peak usage).

Analyzing the NA12878 1000 Genomes longer-read data (2 x 250bp, 63) takes on the order of 215 CPU hours and about 75G of RAM (peak usage).

Of note, negative selection and read-to-PRG alignment are the RAM-intensive steps - positive selection typically doesn't take more than 15G.

## Installation

1. Follow the make process described for MHC\*PRG - you do NOT need the MHC-PRG data package.
2. Download the HLA\*PRG data package (http://birch.well.ox.ac.uk/HLA-PRG.tar.gz) and extract it into ../tmp2/GS_nextGen/hla (relative to the MHC-PRG/src directory, which contains the HLAtypeinference.pl script).
3. Verify that the MHC\*PRG binary is functional by executing ../bin/MHC-PRG from the MHC-PRG/src directory - it is functional if it then complains that you didn't specify a valid mode.
4. Verify that HLAtypeinference.pl is functional by typing perl -c HLAtypeinference.pl - on many systems all required modules will already be present, but might not be the case for you.

Done!

## HLA type inference

HLA type inference is carried out on a per-sample basis.

For each (indexed and sorted) input BAM, you need to specify a unique sample ID. All data associated with this sample will be stored in ../tmp/hla/$SAMPLEID - each sample will consume between 0.5 and 2G of data in ../tmp/hla.

To generate HLA types, you need to execute the following 4 commands from the MHC-PRG/src directory:

./HLAtypeinference.pl --actions p --sampleIDs SAMPLEID --BAMs /path/to/indexed/bam.bam --referenceGenome /path/to/referenceGenome/as/one/fasta/file
./HLAtypeinference.pl --actions n --sampleIDs SAMPLEID
./HLAtypeinference.pl --actions a --sampleIDs SAMPLEID
./HLAtypeinference.pl --actions i --sampleIDs SAMPLEID

You can combine the actions in one command:

./HLAtypeinference.pl --actions pnai --sampleIDs SAMPLEID --BAMs /path/to/indexed/bam.bam --referenceGenome /path/to/referenceGenome/as/one/fasta/file

... or specify multiple BAM files - concatenate the paths with the ',' character (you then also need to specify multiple --sampleIDs).

Some remarks:
- p, n, a, i stand for positive selection, negative selection, alignment, inference.
- if you don't provide explicit sample IDs, the script will try to guess them from the name of the BAM file. It is better to explicitly specify sample IDs.
- the --referenceGenome parameter must point to a single fasta file that contains the reference genome used for creating the input BAM files.

If the inference process was successful, output best-guess HLA types are in ../tmp/hla/$SAMPLEID/R1_bestguess.txt. There is a separate quality score associated with each allele, but our evaluations show that these are not very well-calibrated. The algorithm that extracts two best-guess alleles from a list of allele pairs with associated probabilities is identical to the one used in HLA*IMP:02. Full allele pair probability distributions for locus $LOCUS can be found in the file R1_PP_$LOCUS_pairs.txt.



