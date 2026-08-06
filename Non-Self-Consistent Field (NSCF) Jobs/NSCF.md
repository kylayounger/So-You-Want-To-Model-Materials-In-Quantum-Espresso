# Non-Self-Consistent Field (NSCF) Jobs

### Table of Contents
1. [NSCF Job Basics](#nscf-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Selecting K-paths]()
5. [Example NSCF Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/nscf.in)

## NSCF Job Basics
**Purpose:** to generate eigenvectors and eigenvalues of the Kohn-Sham Hamiltonian using a denser k-grid. NSCF calculations use the converged SCF charge density *without updating/recalculating it;* this is why it is so important to have an accurate and precise SCF calculation.

Must be run after a fixed SCF job.

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
    mixing_ndim = 16                     ! number of iteration of mixing scheme
    diagonalization = 'cg'               ! type of diagonalization used; 'cg' = conjugate-gradient, slower but more robust than Davidson 
    diago_full_acc = .true.              ! if .true., unoccupied states diagonalized/processed at the same level of accuracy as occupied states
    startingwfc = 'atomic+random'        ! wfcs start with atomic positions and add small randomization; prevents 'loss' of valence states
    startingpot = 'file'                 ! pulls starting potential from tmp/ directory
    electron_maxstep = 500               ! max number of iterations in NSCF cycle
/
ATOMIC_SPECIES
    ATOM  ATOMIC_MASS  FILENAME          ! links to your pseudopotential files 
    ATOM  ATOMIC_MASS  FILENAME          ! all atom types in your cell must have a pseudopotential file listed here

CELL_PARAMETERS angstrom
    x    0.0    0.0                      ! dimensions of your unit cell/supercell (only needed for ibrav = 0)
    0.0    y    0.0
    0.0    0.0    z

ATOMIC_POSITIONS crystal
! insert OPTIMIZED fractional coordinates from RELAXATION job here

K_POINTS {crystal_b}                        
NUM              ! total number of k-points in k-path
  x  y  z  NUM   ! first k-point
  x  y  z  NUM   ! second k-point
  x  y  z  NUM   ! ...so on
  x  y  z   1    ! end point, weight=1
```

### Significant Parameters:
**diagonalization** - the algorithm used to find the eigenvalues and eigenvectors of the Hamiltonian in order to solve the Kohn-Sham equation. One of the core functions of the pw.x package.

Options include:

> 1. Davidson (david): default diagonalization; requires the smallest number of Hamiltonian applications per root. Fast, but less robust than other methods and requires more memory. Best for relax and SCF jobs.
>    
> 2. Conjugate-Gradient (cg): uses a band-by-band approach to sequentially diagonalize the matrix. Slower than Davidson, but more robust and uses less memory. Best for NSCF jobs.
>    
> 3. Parallel Orbital-Updating (parO): designed for improved parallelization on HPCs and GPUs; good for scaling.
>    
> 4. Residual Minimization Method - Direct Inversion in the Iterative State (rmm-davidson, rrm-paro): approximate method that operates from an initial 'guess'. Used to stabilize the SCF loop. Faster than Davidson, but is prone to missing electronic states.


**K_POINTS** - denotes the specific high-symmetry path in reciprocal space that you wish to sample. This is the K-path that will be displayed in your band structure diagram.

Each k-point listing must include it's fractional x,y,z coordinates followed by its **weight parameter**. This is the number of intermediate points generated between the k-points (similar to resolution). The weight of the final k-point is ignored by the program, so can be arbitrarily set to 1. 

More information on K_POINTS and K-Paths can be found in [Selecting K-Paths](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/Selecting%20K-Paths.md#finding-the-right-k-path-for-your-system).

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
### Significant Parameters:
**-pd .true.** - this modifier enables **pencil decomposition** of the 3D Fourier Fast Transform (FFT) grid, as opposed to the default slab decomposition.

In slab decomposition, only one dimension is split from the 3D FTT grid to create 'slabs'; this limits the maximum number of processors to the size of a single grid dimension (ex. in a 3x3x3 grid, you can use a maximum of 3 processors). While slab decomposition is fast, it scales poorly and can fail when faced with more complex systems.

In pencil decomposition, the 3D FTT grid is split into two dimensions to form columns or rods. Pencil decomposition allows more complex 3D FFT grids to be evaluated by splitting the workload across more processor cores (ex. for a 3x3x3 grid, you can use a maximum of $3^2$ (9) processors). Pencil decomposition takes longer and as such requires additional input from the user to get Quantum Espresso to implement.

I recommend using the -pd .true. modifier in all submission scripts after SCF (NSCF, bands, dos, projwfc, phonon, dynmat). 

**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)

