# FGMP v 1.0
Fungal Genomics Mapping Pipeline

Released: 4/8/25

# *** Contents *** #

+ 1. FGMP description ?
+ 2. Installing FGMP
+ 3. File Listing
+ 4. running FGMP
+ 5. Testing FGMP
+ 6. Authors and help
+ 7. Citing FGMP

# ---------------------------------------- #
# 1. FGMP description

FGMP (Fungal Gene Mapping Project) is a bioinformatic pipeline designed to 
provide in an unbiased manner a set of high quality gene models from fungal
genome assembly. The strategy is based on the screening of the genome using a 
set of highly diversified fungal proteins, that is espected to represent a realistic 
snapshot of fungal diversity. This approach is likely to capture homologs from any 
fungal genome. FMP is based on 593 protein markers and 172 genomic DNA markers 
are conserved in fungi.  

A local version of FGMP can be installed on UNIX platforms. The tool requires the
pre-installation of Perl, NCBI BLAST, HMMER, EXONERATE and AUGUSTUS. 

The pipeline uses information from the selected genes of 40 fungi by first using TBLASTN to 
identify candidate regions in a new genome. Gene structures are delinated using EXONERATE and AUGUSTUS
and validated using HMMER. At the end FGMP produces a set of best predictions and an estimation of the genome 
completeness of the genome analyzed. 

FGMP use NHMMER to screen the genome with ultraconserved DNA markers. It is also possible to search 
protein markers directly in the unassembled reads. Reads are randomly sampled using reservoir sampling
algorithm and screen using BLASTx.

FGMP source code and documentation are available under the GNU GENERAL PUBLIC LICENSE.

# ---------------------------------------- #
# 2. running FGMP 

The FGMP distribution includes several directories and files. Source codes
and documentation are provided in the distribution. To decompress 
simply enter the following command in the shell: 

clone from https://github.com/stajichlab/FGMP 

or

wget -np biocluster.ucr.edu/~ocisse/manuscript/FGMP.v.1.0.tar.gz

tar -xvzf FGMP.v.1.0.tar.gz

The directory 'fgmp_v1.0' will be created in your current directory. 

FGMP requires the pre-installation of the following software: 

- Perl 5 (tested with the version 20)
- BioPerl-1.6.924 
- HMMER v3.0	http://hmmer.janelia.org/
- NCBI BLASTALL (tested using version 2.2.31+) 
- exonerate (tested using version 2.2.0)
- augustus (tested using version 3.0.3)


# ---------------------------------------- #
# 3. Files listing

The FGMP distribution includes the following files and directories:

- data/				Proteins, profiles and cutoff.
- lib/				Perl modules
- sample/			DNA and protein fasta file to test FGMP.
- sample_output/	Results provided by FGMP for the test files.
- src/				Source code of FGMP.	
- READE.md			This file.
- fgmp.config		a file that needs to be set up with path directories
- utils/			contains useful scripts (see below)

NOTE: augustus distribution should be placed in utils/

example: FGMP.v.1.0/1.0/utilsaugustus-3.0.3


# ---------------------------------------- #
# 4. running FGMP

**** IMPORTANT ****

IMPORTANT - the 'fgmp.config' file needs to be placed where are the fasta files

In the file 'fgmp.config' contains several environmental variables
that need to be set up by the user.

It contains the following variables:

FGMP	- the path to the FGMP folder
WRKDIR	- working directory	(where are the fasta files)

For example: 

FGMP=/bigdata/ocisse/Project3_cema/Version1/src/fgmp/1.0

WRKDIR=/bigdata/ocisse/Project3_cema/Version1/src/fgmp/1.0/sample

FGMP uses a custom library Fgmp.pm. You need set the PERL5LIB environment variable 
to use or you can simply copy the modules to the Perl module directory that is 
available to your Perl installation.

Before running FGMP, type the following commands:
	
	export FGMP=/path/to/fgmp
	export PERL5LIB="$PERL5LIB:$FGMP/lib"

	example:
	export FGMP=/rhome/ocisse/ocisse/tmp/FGMP.v.1.0/1.0
	export PERL5LIB="$PERL5LIB:$FGMP/lib"


To use FGMP with default settings run:
	fgmp.pl -g < genomic_fasta_file > 

You can specify the number of cpus to use using the -T option, which will be passed
to all subsequent softwares.

	-p, --protein		fasta file of the protein sequence
				(default: $FGMP/data/593_cleanMarkers.fa)

	--fuces_hmm		Directory that contains hmm files
				(default: $FGMP/data/172_fUCEs.hmm)

	-c, --cutoff_file	Profile cutoffs
				(default: $FGMP/data/profiles_cutOff.tbl)
	--hmm_profiles		Directory that contains hmm files
				(default: $FGMP/data/593_cleanMarkers.hmm)
	

Example:	perl $FGMP/src/fgmp.pl -g Ncrassa_V2.fasta -T $CPU -r sd_merge.fq.fasta


If you have another set of proteins you want use instead, simply provide them. 


	
# ---------------------------------------- #
# 5. TESTING FGMP

launch the following command and compare the output with the sample files in 'sample_output'

	 perl $FGMP/src/fgmp.pl -g sample.dna 2> log

	 note: the configuration file 'fgmp.config' need to be placed in the current directory 

# running on cluster (example)
	
	#PBS -l nodes=1:ppn=4,mem=8gb -N sample_fg -j oe -l walltime=200:00:00

	CPU=6

	if [ $PBS_NUM_PPN ]; then
 		CPU=$PBS_NUM_PPN
	fi

	export FGMP=/path/to/fgmp/1.0
	export PERL5LIB="/bigdata/stajichlab/ocisse/Project3_cema/Version1/src/fgmp/1.0/lib:$FGMP/lib"

	cd /fgmp/1.0/sample

	perl $FGMP/src/fgmp.pl \
	-g Mycosphaerella_graminicola_IPO323.Mycgr3.v2.fasta \
	-T $CPU

	FGMP will create some intermediate files during the annotation.

	# - final files:
	- sample.dna.bestPreds.fas	: predicted best predictions (fasta format)
	- sample.dna.unfiltered.renamed.hmmsearch.full_report: detailed analysis of best predictions
	- sample.dna.unfiltered.renamed.hmmsearch.summary_report: summary

	# - intermediate files: 
	- sample.dna.tblastn: 	tblastn output
	- sample.dna.candidates.fa: Genomic regions extracted based on Tblastn matches coordinates (fasta format)
	- sample.dna.candidates.fa.p2g: Alignment of 593 proteins to candidates.fa
	- sample.dna.candidates.fa.p2g.aa: exonerate alignment matches
	- sample.dna.candidates.fa.p2g.aa.proteins: translated CDS (amino acids)
	- sample.dna.trainingSet: augustus training set
	- sample.dna.trainingSet.gb: augustus training set (genbank format)
	- sample.dna.unfiltered: unfiltered predicted peptides
	- sample.dna.unfiltered.renamed : renamed predicted peptides to avoid name conflits
	- sample.dna.unfiltered.renamed.hmmsearch : Hmmsearch output


# ---------------------------------------- #
# 6. AUTHORS AND HELP

FGMP has been developped by Ousmane H. Cisse (ousmanecis@gmail.com) and Jason E. Stajich 
(jason.stajich@ucr.edu).
 
FGMP home page is at https://github.com/stajichlab/FGMP

# ---------------------------------------- #
# 7. Citing FGMP

OHC AND JES (2016) FGMP: assessing genome completion in fungal genomic data. manuscript in preparation.
