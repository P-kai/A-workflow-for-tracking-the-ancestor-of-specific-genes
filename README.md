[TOC]
# Large-scale bacterial genomic and metagenomic analysis reveals Pseudomonas aeruginosa as potential ancestral source of tigecycline resistance gene cluster tmexCD-toprJ

## The workflow for the analyses of bacterial genomes and long-read metagenomes:

### Download the bacterial genomes using "ncbi-genome-download-runner.py"
#Tool link:https://github.com/kblin/ncbi-genome-download  
#Prepare the assembly-accessions of the genomes to be downloaded  
#For example: assembly-accessions.txt

GCA_900585555.1  
GCA_900585565.1  
GCA_900585575.1   
GCA_900585585.1   


``ncbi-genome-download-runner.py -s genbank -F fasta -A assembly-accessions.txt -o output_floder bacteria --flat-output``

### Download the genomic information using "bannal.py" in bannal.zip.
#The usage of "bannal.py".
``python bannal.py -h``

``bannel is a program for downloading genome or sample information``  
``     --geo_download input a key word``  
``                    example:  bannel.py --geo_download 'Escherichia'``  
``     --inf_download input a file with assembly_accession number in lines``  
``                    example:  bannel.py --inf_download SAM_list.tab``  
``     -o output file location``  
``     --get_db    get database information of NCBI genbank``  
``                 please run it bannal in first run``  

### Download the metagenomic datasets using "wget"
#The metagenomic datasets were downloaded using wget based on their sra-link.  
#For example:  
``wget https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos5/sra-pub-run-32/ERR002/2581/ERR2581098/ERR2581098.2``

### Assess the quality of bacterial genomes using "checkm"
#The bacterial genomes are stored under the folder bacterial_genomes  
#-x, --extension EXTENSION extension of bins (other files in folder are ignored) (default: fna)  
``checkm taxonomy_wf domain Bacteria bacterial_genomes/ checkm -x fa``  

### Identification of tmexCD-toprJ in bacterial genomes using abricate
#We created a database that included only tmexCD1-toprJ1 sequence:tmexCD-toprJ.  
#Using abricate to identify the tmexCD-toprJ.    
#*fasta includes all bacterial genomes.  
#abricate_output.tab is the output result.  
``abricate --db tmexCD-toprJ --minid 70 --mincov 80 *fasta > abricate_output.tab``  

### Extracting sequences using seqkit according to contig’s ID
#Prepare the sequences' ID of the contigs.  
#For example the file sequence_ID:  
``LR813085.1``  
``LR813086.1``  
#Using seqkit to extract sequences  
``seqkit grep -f sequence_ID bacteria_genome.fasta > extracted_sequences.fasta``  

### Constructing a phylogenetic tree of different tmexCD-toprJ and mexCD-oprJ variants using "muscle" and "RAxML".
#Generate alignment files for the different variants.  
``muscle -in tmexCD-toprJ_variants.fa -out tmexCD-toprJ_variants.aln``  
#Create a phylogenetic tree.  
``raxml-ng --search1 --msa tmexCD-toprJ_variants.aln --model GTR+G``  

### Constructing a phylogenetic tree of different tmexCD-toprJ-positive strains using "prokka", "Roary" and "Fasttree".
#Using Prokka annotate the bacterial genomes.  
``prokka bacteria_genome.fasta --outdir annotation/ --prefix bacteria_genome --kingdom Bacteria``  
#Generate core gene alignment from gff files using roary.  
``roary -f output_dir -e -n -v ./gff/*.gff``  
#Create a phylogenetic tree using fasttree.  
``fasttree -nt -gtr core_gene_alignment.aln > mytree``
