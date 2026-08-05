# Non-Self-Consistent Field (NSCF) Jobs

### Table of Contents
1. [NSCF Job Basics](#nscf-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)

## NSCF Job Basics
**Purpose:** 

**Package:** pw.x

**Resource/Time Usage:** medium/high

An example [NSCF input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/nscf.in) is attached for your reference.

## Input File Structure

```
&CONTROL
    calculation = 'bands'
    prefix = 'NAME'                      ! must be the same name as FIXED
    outdir = './tmp'                     ! must be the same outdir as FIXED
    pseudo_dir = '/home/kylay/scratch/PAW_BTO/psueds'
    restart_mode = 'from_scratch'
    tprnfor = .true.                     ! calculates forces (not sure if this is required)
/
&SYSTEM
    ibrav = 0
    nat = 40
    ntyp = 3
    ecutwfc = 60
    ecutrho = 600
    occupations = 'fixed'             
    nbnd = 200                       ! must be the same as SCF
/
&ELECTRONS
    conv_thr = 1.0d-8
    mixing_mode = 'plain'
    mixing_ndim = 16
    diagonalization = 'cg'
    diago_full_acc = .true.
    startingwfc = 'atomic+random'
    startingpot = 'file'
    electron_maxstep = 500
/
ATOMIC_SPECIES
! FILL IN !

CELL_PARAMETERS angstrom
! FILL IN !

ATOMIC_POSITIONS crystal
! FILL IN !

K_POINTS {crystal_b}                        !IMPORTANT!
8
  0.000  0.000  0.000  30   ! Gamma
  0.500  0.000  0.000  30   ! X
  0.500  0.500  0.000  30   ! M
  0.000  0.000  0.000  30   ! Gamma
  0.000  0.000  0.500  30   ! Z
  0.500  0.000  0.500  30   ! R
  0.500  0.500  0.500  30   ! A
  0.000  0.000  0.500   1   ! Z (end point, weight=1)
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

echo "Starting NSCF: $(date)"        ! written to .out file
export OMP_NUM_THREADS=1             ! MPI threading
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} -pd .true. < nscf.in >> nscf.out     ! run line

echo "Completed NSCF: $(date)"       ! written to .out file
```
