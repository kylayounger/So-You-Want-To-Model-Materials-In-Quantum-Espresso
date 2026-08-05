# Phonon Jobs

### Table of Contents
1. [Phonon Job Basics](#phonon-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)

## Phonon Job Basics

## Input File Structure

```

```

## Submission File Structure

```
#!/bin/bash
#SBATCH --job-name=NAME               ! this name will appear in the queue
#SBATCH --account=ACCT                ! links to resource allocation
#SBATCH --nodes=1                     ! number of computer nodes requested
#SBATCH --ntasks-per-node=192         ! number of CPUs in requested node
#SBATCH --time=00:30:00               ! IMPORTANT
#SBATCH --mem=0                       ! memory allocation, 0 = unlimited
#SBATCH --output=%x_%j.out            ! writes an additional output file
#SBATCH --error=%x_%j.err             ! writes an error file

VERBOSE=TRUE                          ! ensures output files are as detailed as possible
module --force purge                  ! clears previous modules and loads in the necessary ones
module load StdEnv/2023
module load quantumespresso/7.5

cd ${SLURM_SUBMIT_DIR}
mkdir -p tmp                         ! creates tmp/ directory

NTASKS=${SLURM_NTASKS}
NPOOL= NUM                           ! how tasks are divided up between nodes/CPUs

echo "Starting PH.X: $(date)"        ! writes to .out file
export OMP_NUM_THREADS=1             ! MPI threading
mpirun -np ${NTASKS} ph.x -npool ${NPOOL} < ph.in >> ph.out       ! run line

echo "Completed PH.X: $(date)"       ! writes to .out file
```
