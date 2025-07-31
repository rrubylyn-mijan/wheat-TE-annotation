# Wheat Repeat Annotation with RepeatMasker and Custom TE Library

This repository documents the pipeline used to annotate transposable elements (TEs) in wheat using **RepeatMasker v4.2.0** with a custom library combining sequences from **ClariTeRep** and **TREP**.

---

## Installation and Setup (on CCAST)

```bash
# --- Step 1: Download RepeatMasker ---
wget https://www.repeatmasker.org/RepeatMasker/RepeatMasker-4.2.0.tar.gz
tar -xzvf RepeatMasker-4.2.0.tar.gz
cd RepeatMasker

# --- Step 2: Clone and Build TRF from GitHub ---
git clone https://github.com/Benson-Genomics-Lab/TRF.git
cd TRF
mkdir build 
cd build
../configure
make
# Output binary: ./src/trf
# Example path to use: /mmfs1/scratch/first.last/repeat-annotation/TRF/build/src/trf

# --- Step 3: Install rmblast ---
wget https://www.repeatmasker.org/rmblast/rmblast-2.14.1+-x64-linux.tar.gz
tar -xzvf rmblast-2.14.1+-x64-linux.tar.gz
# Example binary path: /mmfs1/scratch/first.last/repeat-annotation/rmblast-2.14.0/bin/

# --- Step 4: Configure RepeatMasker ---
cd ../RepeatMasker
perl ./configure
# Provide full paths to rmblast, TRF, and your Perl interpreter when prompted

# --- Step 5: Optional - Convert genome to 2bit (not required for RepeatMasker) ---
wget http://hgdownload.soe.ucsc.edu/admin/exe/linux.x86_64/faToTwoBit
chmod +x faToTwoBit
./faToTwoBit /mmfs1/home/rubylyn.infante/repeat-annotation/Sumai3_pm_v2.fasta Sumai3_pm_v2.2bit

# --- Step 6: Prepare Custom TE Library ---
git clone https://github.com/jdaron/CLARI-TE.git
cd CLARI-TE
wget http://botserv2.uzh.ch/kelldata/trep-db/blast/dir_download/sequences.fasta
cat clariTeRep.fna sequences.fasta > 1custom_wheat_TE_library.fasta
```

## Run RepeatMasker (PBS Job Script)
```
#!/bin/bash
#PBS -q bigmem
#PBS -N full_repeat_annotation_sumai3_clarite_library_1
#PBS -l select=1:mem=1000gb:ncpus=48
#PBS -l walltime=96:00:00
#PBS -M first.last@ndsu.edu
#PBS -m abe
#PBS -W group_list=x-ccast-prj-name

cd /mmfs1/scratch/first.last/repeat-annotation/RepeatMasker

mkdir -p logs 3full_repeat_annotation

./RepeatMasker \
  -pa 48 \
  -e ncbi \
  -lib /mmfs1/scratch/first.last/repeat-annotation/CLARI-TE/1custom_wheat_TE_library.fasta \
  -xm \
  -a \
  -gff \
  -xsmall \
  -dir 3full_repeat_annotation \
  Sumai3_pm_v2.fasta \
  2>&1 | tee logs/04_simplemask.log
```
