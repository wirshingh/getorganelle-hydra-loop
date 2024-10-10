# Run getorganelle on hydra using a loop
### Summary
This scritp will run getorganelle on hydra using trimmed reads in a loop. Results for all samples will be in a directory named 'getorganelle_All_results'. Within this directory will be results for each sample in directories labeled with sample IDs followed by '_getorganelle_results'.

After getorganelle is run, the script contains commands that will copy the final mitochondrial contig (or scaffolds) and rename it with sample IDs. These renamed contigs will be in a directory named 'mt_contigs' within the 'getorganelle_All_results' directory.

For instuctions on how to run the job, see 'To Run the Job' below.


```
#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 6
#$ -q sThC.q
#$ -l mres=48G,h_data=8G,h_vmem=64G
#$ -cwd
#$ -j y
#$ -N getorg
#$ -o getorganelle.log
#
# ----------------Modules------------------------- #
module load bio/getorganelle
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS

# Create output directory if it doesn't exist
mkdir -p getorganelle_All_results

# Set sample directory path to trimmed reads
SAMPLEDIR_TRM="path to trimmed reads directory"

# Set sample directory path to the base project directory
SAMPLEDIR_BASE="path to base project directory, where the job file is."  

# Make a results directory
mkdir -p ${SAMPLEDIR_BASE}/getorganelle_All_results/

# Loop over each R1 file matching the pattern
for GETSAMPLENAME in ${SAMPLEDIR_TRM}/*_R1_PE_trimmed.fastq.gz; do
SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)

# Run getorganelle
get_organelle_from_reads.py \
-1 "${SAMPLEDIR_TRM}/${SAMPLENAME}_R1_PE_trimmed.fastq.gz" \
-2 "${SAMPLEDIR_TRM}/${SAMPLENAME}_R2_PE_trimmed.fastq.gz" \
-u "${SAMPLEDIR_TRM}/${SAMPLENAME}_R0_SE_trimmed.fastq.gz" \
-o "${SAMPLEDIR_BASE}/getorganelle_All_results/${SAMPLENAME}_getorganelle_results" \
-F animal_mt \
--disentangle-time-limit 7200 \
-t $NSLOTS
done

# Extra steps to rename output files with sample names
# Create a directory for renamed output files
mkdir -p ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs

# Loop over each R1 file matching the pattern
for GETSAMPLENAME in ${SAMPLEDIR_TRM}/*_R1_PE_trimmed.fastq.gz; do
SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)

# Copy final mitochondrial contigs to a new directory called "mt_contigs"
cp ${SAMPLEDIR_BASE}/getorganelle_All_results/${SAMPLENAME}_getorganelle_results/*.path_sequence.fasta ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs

# Rename copied mitochondrial contigs with sample name and "_mt_contigs.fasta"
mv ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/*.path_sequence.fasta ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/${SAMPLENAME}_mt_contigs.fasta
done

# Rename internal text of mitochondrial contigs with sample name after the > in fasta files
# Loop through each renamed fasta file in the 'mt_contigs' directory
for input_file in ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/*.fasta; do
    # Extract the filename without the extension
    filename=$(basename "$input_file" .fasta)
    
    # Create the output file name
    renamed_contigs="${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/${filename}_renamed.fasta"
    
    # Process the file and insert the filename after each '>'
    awk -v fname="$filename" '{gsub(/>/, ">" fname)}1' "$input_file" > "$renamed_contigs"
    
done

# Remove old files
rm ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/*_contigs.fasta

echo "All files renamed and processed."

echo = `date` job $JOB_NAME

```

### To Run the Job
The trimmed reads files must end in '_R1_PE_trimmed.fastq.gz' (forward) and '_R2_PE_trimmed.fastq.gz' (reverse) for the job to work. Alternatively, the job file can be edited to match the trimmed R1 and R2 file names.

Two items need to be added for the job to run.

1. SAMPLEDIR_TRM="path to trimmed reads directory"

After the '=' paste the full path to the trimmed reads directory.

2. SAMPLEDIR_BASE="path to base project directory, where the job file is."

After the '=' paste the full path to base directory. This directory is where the job file is located. The results will be located in this directory.

After making the changes above, save the job file as 'getorganelle_loop.job' and submit it on hydra (qsub getorganelle_loop.job).




