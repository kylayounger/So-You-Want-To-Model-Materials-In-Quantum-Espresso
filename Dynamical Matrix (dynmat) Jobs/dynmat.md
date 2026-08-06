# Dynamical Matrix (dynmat) Jobs

### Table of Contents
1. [Dynmat Job Basics](#dynmat-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Example Dynmat Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Dynamical%20Matrix%20(dynmat)%20Jobs/dynmat.in)

## Dynmat Job Basics
**Purpose:** to post-process data generated in a phonon job and apply an Acoustic Sum Rule (ASR). Diagonalizes the phonon matrix to yield properties such as Raman and IR tensors, vibrational modes, and intensities at the gamma point.

Must be performed after a Phonon job.

**Package:** dynmat.x

**Resource/Time Usage:** low

An example [dynmat input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Dynamical%20Matrix%20(dynmat)%20Jobs/dynmat.in) is attached for your reference (2x2x2 Tetragonal Barium Titanate Supercell).

## Input File Structure
```
&INPUT
   fildyn = 'NAME'               ! labels and collects data associated with this job, MUST BE THE SAME as directory containing wavefunctions
   asr = 'crystal'               ! type of acoustic sum rule used; 'crystal' = 3-translational asr applied by correction of dynamical matrix
   filout = 'NAME_dynmat.out'    ! name of dynmat.out file
   filmol = 'NAME_dynmat.mold'   ! name of dynmat.mold file
   filxsf = 'NAME_dynmat.xsf'    ! name of dynmat.xsf file
/
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

echo "Starting DYNMAT: $(date)"      ! written to .out file
export OMP_NUM_THREADS=1             ! MPI threading
mpirun -np ${NTASKS} dynmat.x -npool ${NPOOL} -pd .true. < dynmat.in >> dynmat.out       ! run line

echo "Completed DYNMAT: $(date)"     ! written to .out file
```
### Significant Parameters:
**-pd .true.** - see [NSCF Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/NSCF.md#significant-parameters-1)

**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)
