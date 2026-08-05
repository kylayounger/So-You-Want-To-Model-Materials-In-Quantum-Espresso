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
    calculation = 'bands'                ! designates NSCF as the type of job you want to run
    prefix = 'NAME'                      ! labels and collects data associated with this job, must be the same name as SCF
    outdir = './tmp'                     ! where wavefunctions and other data is stored, must be the same as SCF
    pseudo_dir = 'path/to/pseuds'        
    restart_mode = 'from_scratch'        !'from_scratch' = completly restarts, 'restart' = continue from where last job finished
    tprnfor = .true.                     ! calculates forces
/
&SYSTEM
    ibrav = 0                            ! cell type; must be the same as SCF
    nat = NUM                            ! total number of atoms in your base cell/supercell
    ntyp = NUM                           ! total number of elements in your system
    ecutwfc = NUM                        ! kinetic energy cutoff for wfcs
    ecutrho = NUM                        ! kinetic energy cutoff for charge density, 4*ecutwfc is default
    occupations = 'fixed'                ! 'smearing' = smeared electrons for metals, 'fixed' = insulator with a bandgap
    nbnd = NUM                           ! total number of electronic bands to be calculated, must be the same as SCF
/
&ELECTRONS
    conv_thr = 1.0d-NUM                  ! can be slightly decreased from fixed SCF
    mixing_mode = 'plain'                ! type of electron mixing
    mixing_ndim = 16                     
    diagonalization = 'cg'               ! type of diagonalization used; 
    diago_full_acc = .true.              !
    startingwfc = 'atomic+random'        !
    startingpot = 'file'                 ! pulls starting potential from tmp/ directory
    electron_maxstep = 500               ! max number of iterations in NSCF cycle
/
ATOMIC_SPECIES
    ATOM  ATOMIC_MASS  FILENAME     ! links to your pseudopotential files 
    ATOM  ATOMIC_MASS  FILENAME     ! all atom types in your cell must have a pseudopotential file listed here

CELL_PARAMETERS angstrom
    x    0.0    0.0                 ! dimensions of your unit cell/supercell (only needed for ibrav = 0)
    0.0    y    0.0
    0.0    0.0    z

ATOMIC_POSITIONS crystal
! insert OPTIMIZED fractional coordinates from RELAXATION job here

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
