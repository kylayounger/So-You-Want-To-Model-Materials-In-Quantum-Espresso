# Phonon Jobs

### Table of Contents
1. [Phonon Job Basics](#phonon-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)

## Phonon Job Basics

## Input File Structure

```
&INPUTPH
    outdir = './tmp'         ! Must be same outdir as FIXED SCF
    prefix = 'NAME'          ! Must be same prefix as FIXED SCF
    fildyn = 'NAME'
    fildvscf = 'dvscf'
    tr2_ph = 1.0d-12         ! Self-consistency threshold; preset to 1.0d-12, ideally 1.0d-14
    epsil = .true.           ! .true. = computes dielectric tensor (only works for non-metal systems)
    ldisp = .false.          ! .true. = calculates phonons for specified k-point grid (not needed here)
    recover = .true.         ! This is the restart line, .true. = restart, .false. = no restart
    only_init = .false.      ! .true. = only bands and initialization quantities calculated on restart
/
0.0 0.0 0.0
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
mpirun -np ${NTASKS} ph.x -npool ${NPOOL} -pd .true. < ph.in >> ph.out       ! run line

echo "Completed PH.X: $(date)"       ! writes to .out file
```
### Significant Parameters:
**-pd .true.** - see [NSCF Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/NSCF.md#significant-parameters-1)

**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)
