# Run MITOS on Hydra in a loop
### Summary
This script will run MITOS on Hydra using assembled mitochondrial contigs in '.fasta' format.

The final results (with the most important files) will be copied and renamed to the directory named 'mitos_renamed_results'.
All of the outputed results will be in a directory named 'mitos_All_results'. 

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
mkdir -p ${SAMPLEDIR_BASE}/mitos_renamed_results

for GETSAMPLENAME in ${SAMPLEDIR_MTCONTIG}/*.fasta; do
    SAMPLENAME=$(basename "$GETSAMPLENAME" .fasta)

    mkdir -p mitos_renamed_results/${SAMPLENAME}

    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.bed ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.bed
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.faa ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.faa
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.fas ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.fas
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.geneorder ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.geneorder
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.gff ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.gff
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*ignored.mitos ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}_ignored.mitos
     cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*result.mitos ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}_result.mitos
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.png ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.png
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.seq ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.seq
    cp ${SAMPLEDIR_BASE}/mitos_All_results/${SAMPLENAME}_mitos_results/*.dat ${SAMPLEDIR_BASE}/mitos_renamed_results/${SAMPLENAME}/${SAMPLENAME}.dat
done
echo "DONE. Final renamed MITOS results copied to 'mitos_renamed_results'"
echo = `date` job $JOB_NAME

```

### To Run the Job
The user must eddit two items in the job file above:

1. SAMPLEDIR_MTCONTIG="path to assembled mt contigs. Must end in '.fasta'"

After the '=' paste the full path to the directory with mitochondrial contigs in '.fasta' format.

2. SAMPLEDIR_BASE="path the base project directory. Directory where job file is"

After the '=' paste the full path to base directory. This directory is where the job file is located. The results will be located in this directory.

After making the changes above, save the job file as 'mitos_loop.job' and submit it on hydra (qsub mitos_loop.job).
