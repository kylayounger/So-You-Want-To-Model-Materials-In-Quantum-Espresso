# Average Jobs
### Table of Contents
1. [Average Job Basics](#average-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Example Average Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Average%20Jobs/average.in)

## Average Job Basics
**Purpose:** to process the output of PP runs.

**Package:** average.x

**Resource/Time Usage:** low

An example [Average input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Average%20Jobs/average.in) is attached for your reference (2x2x2 Tetragonal Barium Titanate).

## Input File Structure
```
1                      ! Number of files to read in
total_pot.dat          ! Name of file to be read in (from pp.x)
1.0                    ! Weight assigned to file
200                    ! Number of interpolation points along axis
3                      ! Direction of vacuum axis (1 = X, 2 = Y, 3 = Z)
0.0                    ! Size of macroscopic avergaing window (0.0 = standard planar average)
```

## Submission File Structure
```
#!/bin/bash
#SBATCH --job-name=NAME             ! this name will appear in the queue
#SBATCH --account=ACCT              ! links to resource allocation
#SBATCH --nodes=1                   ! number of computer nodes requested
#SBATCH --ntasks=192                ! number of CPUs in requested node
#SBATCH --cpus-per-task=1
#SBATCH --time=00:30:00             ! IMPORTANT
#SBATCH --mem=0                     ! memory allocation, 0 = unlimited
#SBATCH --output=%x_%j.out          ! writes an additional output file
#SBATCH --error=%x_%j.err           ! writes an error file

VERBOSE=TRUE                        ! ensures output files are as detailed as possible
module --force purge                ! clears previous modules and loads in the necessary ones
module load StdEnv/2023
module load quantumespresso/7.5

cd ${SLURM_SUBMIT_DIR}
mkdir -p tmp                       ! creates tmp/ directory

NTASKS=${SLURM_NTASKS}
NPOOL= NUM                         ! how tasks are divided up between nodes/CPUs

echo "Starting AVERAGE: $(date)"       ! written to .out file
export OMP_NUM_THREADS=1               ! MPI threading
mpirun -np ${NTASKS} average.x -npool ${NPOOL} -pd .true. < average.in >> average.out      ! run line

echo "Completed AVERAGE: $(date)"      ! written to .out file
```
