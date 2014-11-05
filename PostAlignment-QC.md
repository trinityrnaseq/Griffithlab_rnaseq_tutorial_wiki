#POST-ALIGNMENT QC
Use samtools, samstat, and FastQC to evaluate the alignments
Use 'samtools view' to see the format of a SAM/BAM alignment file
	cd $RNA_HOME/alignments/tophat/Normal_ALL
	samtools view -H accepted_hits.bam
	samtools view accepted_hits.bam | head
	
Try filtering the BAM file to require or exclude certain flags
This can be done with samtools view -f -F options
 -f INT   required flag
 -F INT   filtering flag
Google "samtools flags explained"
 http://picard.sourceforge.net/explain-flags.html
Try requiring that alignments are 'paired' and 'mapped in a proper pair' (=3)
ALso filter out alignments that are 'unmapped', the 'mate is unmapped', and 'not primary alignment' (=268)
	samtools view -f 3 -F 268 accepted_hits.bam | head
	
Now require that the alignments be only for 'PCR or optical duplicate'
How many reads meet this criteria? Why?
	samtools view -f 1024 accepted_hits.bam | head
	
Use 'samtools flagstat' to get a basic summary of an alignment
What percent of reads are mapped? Is this realistic? Why?
	cd $RNA_HOME/alignments/tophat/
	samtools flagstat Normal_cDNA1_lib2/accepted_hits.bam
	samtools flagstat Normal_cDNA2_lib2/accepted_hits.bam
	
Run samstat on Tumor/Normal BAMs
	cd $RNA_HOME/alignments/tophat/
	samstat Normal_ALL/accepted_hits.bam
	samstat Tumor_ALL/accepted_hits.bam
	
View the samstat summary file in a web browser
Note, you must replace cbw** with your own amazon instance number (e.g., "cbw01"))
 http://cbw##.ssh01.com/rnaseq/alignments/tophat/Normal_ALL/accepted_hits.bam.html
 http://cbw##.ssh01.com/rnaseq/alignments/tophat/Tumor_ALL/accepted_hits.bam.html
	
Details of the SAM/BAM format can be found here
 http://samtools.sourceforge.net/SAM1.pdf

##Using FastQC
You can use FastQC (reference above) to perform basic QC of your accepted_hits.bam file
This will give you output very similar to when you ran FastQC on your fastq files.
	
	
	