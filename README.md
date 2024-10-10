# Run MITOS on Hydra in a loop
### Summary
This script will run MITOS on Hydra using assembled mitochondrial contigs in '.fasta' format.

Results will be in a directory named 'mitos_All_results'. Within this directory results for each sample will be in its own directory with the sample ID and end in '_mitos_results'.

To prepare the job file, see 'To Run the Job' below.
```

#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q sThM.q
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
echo = `date` job $JOB_NAME

```

### To Run the Job
The user must provide two items.

1. SAMPLEDIR_MTCONTIG="path to assembled mt contigs. Must end in '.fasta'"

After the '=' paste the full path to the directory with mitochondrial contigs in '.fasta' format.

2. SAMPLEDIR_BASE="path the base project directory. Directory where job file is"

After the '=' paste the full path to base directory. This directory is where the job file is located. The results will be located in this directory.

After making the changes above, save the job file as 'mitos_loop.job' and submit it on hydra (qsub mitos_loop.job).
