# Run MITOS on Hydra in a loop
### Summary
This script will run MITOS on Hydra using assembled mitochondrial contigs in '.fasta' format.

All results will be in a directory named 'mitos_All_results'. 
The most important files will be copied and renamed with sample names to the directory 'mitos_renamed_results'.


To prepare the job file, see 'To Run the Job' below.
```

#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q mThM.q
#$ -l mres=48G,h_data=48G,h_vmem=48G,himem
#$ -cwd
#$ -j y
#$ -N mitos
#$ -o mitos.log
#
# ----------------Modules------------------------- #
module load bio/mitos
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#
mkdir -p mitos_All_results

# Set path to directory with assembled mitochondrial contigs
SAMPLEDIR_MTCONTIG="path to assembled mt contigs. Must end in '.fasta'"

# Set path to base project directory
SAMPLEDIR_BASE="path the base project directory. Directory where job file is"
#
for GETSAMPLENAME in ${SAMPLEDIR_MTCONTIG}/*.fasta
do
SAMPLENAME=$(basename "$GETSAMPLENAME" .fasta)
#
mkdir -p ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results
#
runmitos.py \
-i ${SAMPLEDIR_MTCONTIG}/${SAMPLENAME}*.fasta \
-c 5 \
-o ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results \
-r /scratch/nmnh_lab/macdonaldk/ref/refseq89m \
-R /scratch/nmnh_lab/macdonaldk/ref/refseq89m \
--debug
done
#
#
# PART2 - Copy and Rename Output files wth sample names
#
# Output directory
OUTPUT_DIR="${SAMPLEDIR_BASE}/mitos_renamed_results"
mkdir -p "$OUTPUT_DIR"

# File extensions to process (excluding .dat)
EXTENSIONS=("bed" "faa" "fas" "geneorder" "gff" "png" "seq")
SPECIAL_EXTENSIONS=("ignored.mitos" "result.mitos")

for GETSAMPLENAME in "$SAMPLEDIR_MTCONTIG"/*.fasta; do
    SAMPLENAME=$(basename "$GETSAMPLENAME" .fasta)
    SAMPLE_OUT_DIR="${OUTPUT_DIR}/${SAMPLENAME}"
    mkdir -p "$SAMPLE_OUT_DIR"

    SAMPLE_SRC_DIR="${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results"

    # Regular extensions (e.g., .bed, .faa, etc.)
    for EXT in "${EXTENSIONS[@]}"; do
        COUNT=1
        find "$SAMPLE_SRC_DIR" -type f -name "*.${EXT}" | while read FILE; do
            BASENAME="${SAMPLENAME}_contig${COUNT}.${EXT}"
            cp "$FILE" "${SAMPLE_OUT_DIR}/${BASENAME}"
            ((COUNT++))
        done
    done

    # Special filenames like *ignored.mitos or *result.mitos
    for EXT in "${SPECIAL_EXTENSIONS[@]}"; do
        COUNT=1
        find "$SAMPLE_SRC_DIR" -type f -name "*${EXT}" | while read FILE; do
            BASENAME="${SAMPLENAME}_contig${COUNT}_${EXT}"
            cp "$FILE" "${SAMPLE_OUT_DIR}/${BASENAME}"
            ((COUNT++))
        done
    done
done

echo "DONE. Final renamed MITOS results copied to '${OUTPUT_DIR}'"
echo = `date` job $JOB_NAME

```

### To Run the Job
The user must eddit two items in the job file above:

1. SAMPLEDIR_MTCONTIG="path to assembled mt contigs. Must end in '.fasta'"

After the '=' paste the full path to the directory with mitochondrial contigs in '.fasta' format.

2. SAMPLEDIR_BASE="path the base project directory. Directory where job file is"

After the '=' paste the full path to base directory. This directory is where the job file is located. The results will be located in this directory.

After making the changes above, save the job file as 'mitos_loop.job' and submit it on hydra (qsub mitos_loop.job).
