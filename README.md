# Run getorganelle on hydra using a loop
### Summary
This scritp will run getorganelle on hydra using trimmed reads in a loop. Results for all samples will be in a directory named 'getorganelle_All_results'. Within this directory will be results for each sample in directories labeled with sample IDs followed by '_getorganelle_results'.

After getorganelle is run, PART 2 of the script contains commands that will copy the final mitochondrial contig (or scaffolds) and rename it with sample IDs. These renamed contigs will be in a directory named 'mt_contigs' within the 'getorganelle_All_results' directory.

For instuctions on how to run the job, see 'To Run the Job' below.


```
#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 6
#$ -q mThC.q
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

# PART 2 - Extra steps to copy and rename output mito contig files with sample names
# Create directory for renamed output files
mkdir -p ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs

# Loop over R1 files
for GETSAMPLENAME in ${SAMPLEDIR_TRM}/*_R1_PE_trimmed.fastq.gz; do
    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)

    # Get the path to the .path_sequence.fasta file
    src_file=$(ls ${SAMPLEDIR_BASE}/getorganelle_All_results/${SAMPLENAME}_getorganelle_results/*.path_sequence.fasta | head -n 1)

    # Copy and rename
    cp "$src_file" ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/${SAMPLENAME}_mt_contigs.fasta
done

# Rename internal headers in fasta files
for input_file in ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/*_mt_contigs.fasta; do
    filename=$(basename "$input_file" .fasta)
    renamed_contigs="${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/${filename}_renamed.fasta"

    # Modify only header lines
    awk -v fname="$filename" '/^>/ {$0=">"fname"_"substr($0,2)} {print}' "$input_file" > "$renamed_contigs"
done

# Remove original unrenamed fasta files
rm ${SAMPLEDIR_BASE}/getorganelle_All_results/mt_contigs/*_mt_contigs.fasta

echo "All files renamed and processed."

echo = `date` job $JOB_NAME

```

### To Run the Job
The trimmed reads files must end in '_R1_PE_trimmed.fastq.gz' (forward) and '_R2_PE_trimmed.fastq.gz' (reverse) for the job to work. Alternatively, the job file can be edited to match the trimmed R1 and R2 file names.

Add these items to the script.

1. SAMPLEDIR_TRM="path to trimmed reads directory"

After the '=' paste the full path to the trimmed reads directory.

2. SAMPLEDIR_BASE="path to base project directory, where the job file is."

After the '=' paste the full path to base directory. This directory is where the job file is located. The results will be located in this directory.

After making the changes above, save the job file as 'getorganelle_loop.job' and submit it on hydra (qsub getorganelle_loop.job).




