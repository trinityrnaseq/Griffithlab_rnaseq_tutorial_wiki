#DIFFERENTIAL EXPRESSION
Use Cuffmerge and Cuffdiff to compare the tumor and normal conditions.
Refer to the Cufflinks manual for a more detailed explanation:
http://cufflinks.cbcb.umd.edu/manual.html#cuffmerge
http://cufflinks.cbcb.umd.edu/manual.html#cuffdiff
	
 Cuffmerge basic usage
 cuffmerge [options]* <assembly_GTF_list.txt> 
 <assembly_GTF_list.txt> is a text file "manifest" with a list (one per line) of GTF files that you'd like to merge together into a single GTF file. 
Extra options specified below
 '-p 8' tells cuffmerge to use eight CPUs
 '-o' tells cuffmerge to write output to a particular directory
 '-g' tells cuffmerge where to find reference gene annotations. It will use these annotations to gracefully merge novel isoforms (for de novo runs) and known isoforms and maximize overall assembly quality.
 '-s' tells cuffmerge where to find the reference genome files
	
Merge all 4 cufflinks results so that they will have the same set of transcripts for comparison purposes
	cd $RNA_HOME/expression/tophat_cufflinks/ref_only/
	ls -1 *cDNA*lib*/transcripts.gtf > assembly_GTF_list.txt
	cuffmerge -p 8 -o merged -g $RNA_HOME/refs/hg19/genes/genes_chr22.gtf -s $RNA_HOME/refs/hg19/bwt/22/ assembly_GTF_list.txt
	
##OPTIONAL ALTERNATIVE
perform the merge step for STAR-alignment-based cufflinks output
	cd $RNA_HOME/expression/star_cufflinks/ref_only/
	ls -1 *cDNA*lib*/transcripts.gtf > assembly_GTF_list.txt
	cuffmerge -p 8 -o merged -g $RNA_HOME/refs/hg19/genes/genes_chr22.gtf -s $RNA_HOME/refs/hg19/bwt/22/ assembly_GTF_list.txt
	
	
 Cuffdiff basic usage
 cuffdiff [options] <transcripts.gtf> <sample1_hits.sam> <sample2_hits.sam> [... sampleN_hits.sam]
 Supply replicate SAMs as comma separated lists for each condition: sample1_rep1.sam,sample1_rep2.sam,...sample1_repM.sam
 '-p 8' tells cuffdiff to use eight CPUs
 '-L' tells cuffdiff the labels to use for samples
	
	cd $RNA_HOME/
	mkdir -p de/tophat_cufflinks/ref_only
	cd $RNA_HOME/alignments/tophat/
	
Perform Tumor vs. Normal comparison, using all tumor replicates vs all normal replicates, for known (reference only mode) transcripts
	cuffdiff -p 8 -L Tumor,Normal -o $RNA_HOME/de/tophat_cufflinks/ref_only/ --frag-len-mean 262 --frag-len-std-dev 80 --no-update-check $RNA_HOME/expression/tophat_cufflinks/ref_only/merged/merged.gtf Tumor_cDNA1_lib2/accepted_hits.bam,Tumor_cDNA2_lib2/accepted_hits.bam Normal_cDNA1_lib2/accepted_hits.bam,Normal_cDNA2_lib2/accepted_hits.bam
	
##OPTIONAL ALTERNATIVE
perform the cuffdiff step for STAR-alignment-based cuffmerge output
	cd $RNA_HOME/
	mkdir -p de/star_cufflinks/ref_only
	cd $RNA_HOME/alignments/star/
	cuffdiff -p 8 -L Tumor,Normal -o $RNA_HOME/de/star_cufflinks/ref_only/ --frag-len-mean 262 --frag-len-std-dev 80 --no-update-check $RNA_HOME/expression/star_cufflinks/ref_only/merged/merged.gtf Tumor_cDNA1_lib2/Aligned.out.sorted.bam,Tumor_cDNA2_lib2/Aligned.out.sorted.bam Normal_cDNA1_lib2/Aligned.out.sorted.bam,Normal_cDNA2_lib2/Aligned.out.sorted.bam
	
What does the raw output from Cuffdiff look like?
	cd $RNA_HOME/de/tophat_cufflinks/ref_only
	ls -l
	head isoform_exp.diff
	grep -P "gene_id|OK" isoform_exp.diff | cut -f 2-6,8-10,12 | sort -k 9,9 | less -S
Press 'q' to exit the 'less' display
	
How many genes are there on this chromosome?
	grep -v gene_id gene_exp.diff | wc -l
	
How many were detected above 0 in Normal or Tumor (take the sum of expression values for both and check for greater than 0)?
	grep -v gene_id gene_exp.diff | perl -ne '@line=split("\t", $_); $sum=$line[7]+$line[8]; if ($sum > 0){print "$sum\n";}' | wc -l
	
How many differentially expressed genes were found on this chromosome (p-value < 0.05)?
	grep -v gene_id gene_exp.diff | cut -f 12 | perl -ne 'if ($_ < 0.05){print "$_"}' | wc -l
	
Display the top 20 DE genes:
Look at some of those genes in IGV - do they make sense?
	grep -P "OK|gene_id" gene_exp.diff | sort -k 12n,12n | head -n 20 | cut -f 3,5,6,8,9,10,12,13,14
	
Save all genes with P<0.05 to a new file
	grep -P "OK|gene_id" gene_exp.diff | sort -k 12n,12n | cut -f 3,5,6,8,9,10,12,13,14 | perl -ne '@data=split("\t", $_); if ($data[6]<=0.05){print;}' > DE_genes.txt
        