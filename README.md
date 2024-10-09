# Run getorganelle on hydra using a loop
This scritp will run getorganelle in hydra using trimmed raw reads in a loop.

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
SAMPLEDIR_TRM="path to trimmed reads"

# Set sample directory path to the base project directory
SAMPLEDIR_BASE="path to base sample directory, where the job file is."  

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


