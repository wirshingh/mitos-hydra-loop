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
SAMPLEDIR_MTCONTIG="full path to assembled mt contigs. Must end in .fasta"

# Set path to base project directory
SAMPLEDIR_BASE="full path the base project directory. Output files will be here"
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

echo "PART 2 DONE. Final renamed MITOS results copied to 'mitos_renamed_results'"

# PART3 - Copy all of the final genes into a single .fasta file for each sample

mkdir -p mitos_Final_Genes

# Loop through all contig1.fas files to get sample names
for GETSAMPLENAME in ./mitos_renamed_results/*contig*/*contig1.fas
do
    # Extract sample name by removing _contig1.fas
    SAMPLENAME=$(basename "$GETSAMPLENAME" _contig1.fas)

    # Define the output file for the sample
    OUTPUT_FILE=./mitos_Final_Genes/${SAMPLENAME}_mitos_Final_Genes.fasta

    # Empty the output file in case it already exists
    > "$OUTPUT_FILE"

    # Concatenate all .fas files for that sample into the output
    for FASFILE in ./mitos_renamed_results/*contig*/${SAMPLENAME}*.fas
    do
        cat "$FASFILE" >> "$OUTPUT_FILE"
    done
done

echo "PART 3 DONE. Final genes for each sample copied to 'mitos_Final_Genes'"

echo = `date` job $JOB_NAME

```

### To Run the Job
The user must the following items in the job file above:

1. SAMPLEDIR_MTCONTIG="path to assembled mt contigs. Must end in '.fasta'"

After the '=' paste the full path to the directory with mitochondrial contigs in '.fasta' format.

2. SAMPLEDIR_BASE="path the base project directory. Directory where job file is"

After the '=' paste the full path to base directory. This directory is where the job file is located. The results will be located in this directory.

3. Genetic Code

After the mitos command flag -c enter the number of the genetic code that will be used for annotation. 5 (Invertebrate Mitochondrial) is set as default.

Genetic codes:

1 The Standard Code 

2 The Vertebrate Mitochondrial Code 

3 The Yeast Mitochondrial Code 

4 The Mold, Protozoan, and Coelenterate Mitochondrial Code and the Mycoplasma/Spiroplasma Code

5 The Invertebrate Mitochondrial Code

6 The Ciliate, Dasycladacean and Hexamita Nuclear Code 

9 The Echinoderm and Flatworm Mitochondrial Code 

10 The Euplotid Nuclear Code 

11 The Bacterial, Archaeal and Plant Plastid Code 

12 The Alternative Yeast Nuclear Code 

13 The Ascidian Mitochondrial Code 

14 The Alternative Flatworm Mitochondrial Code 

16 Chlorophycean Mitochondrial Code 

21 Trematode Mitochondrial Code 

22 Scenedesmus obliquus Mitochondrial Code 

23 Thraustochytrium Mitochondrial Code 

24 Pterobranchia Mitochondrial Code 

25 Candidate Division SR1 and Gracilibacteria Code


After making the changes above, save the job file as 'mitos_loop.job' and submit it on hydra (qsub mitos_loop.job).
