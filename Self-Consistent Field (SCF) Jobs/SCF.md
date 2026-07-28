# Self-Consistent Field (SCF) Jobs

### Table of Contents
1. [SCF Job Basics](#scf-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Key Outputs](#key-outputs)
5. [Smear vs. Fixed Occupations](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear%20vs.%20Fixed%20Occupations.md)
6. [Example Smear SCF Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear.in)
7. [Example Fixed SCF Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Fixed.in)


## SCF Job Basics

**Purpose:** To determine the ground-state energy and electron density of a system. This is accomplished by self-consistently solving the Kohn-Sham equation and constructing discrete electron orbitals. ***Essential for more advanced calculation types.***

When modelling more complex systems (vacancies, double-well potentials, etc), you may need to run *two seperate SCF jobs to reach convergence:* a Smear SCF job and a Fixed SCF job. Specifics about this process are inlcuded below and in [Smear vs. Fixed Occupations](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear%20vs.%20Fixed%20Occupations.md).


**Package:** pw.x

**Resource/Time Usage:** medium/high

An example [Smear SCF input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear.in) and [Fixed SCF input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Fixed.in) are attached for your reference (2x2x2 Tetragonal Barium Titanate Supercell).

## Input File Structure
```
&CONTROL
    calculation = 'scf'             ! designates SCF as the type of job to run
    prefix = 'NAME'                 ! labels and collects data associated with this job
    outdir = './tmp'                ! where wavefunctions and other data is stored (directory must be created before running job)
    pseudo_dir = 'path/to/pseuds'
    tprnfor = .true.
    tstress = .true.
    etot_conv_thr = 1.0d-4  ! Ry
    forc_conv_thr = 1.0d-3  ! Ry/au
    nstep = 500                     ! max number of optimization steps allowed in job
    restart_mode = 'from_scratch'   !'from_scratch' = completly restarts, 'restart' = continue from where last job finished
/
&SYSTEM
    tot_charge = NUM                ! net electronic charge of the cell
    ibrav = 0                       ! cell type; 0 = free cell (determined by CELL_PARAMETERS)
    nat = NUM                       ! total number of atoms in your base cell/supercell
    ntyp = NUM                      ! total number of elements in your system
    ecutwfc = NUM                   ! kinetic energy cutoff for wfcs
    ecutrho = NUM                   ! kinetic energy cutoff for charge density, 4*ecutwfc is default
    occupations = 'smearing'        ! 'smearing' = smeared electrons for metals, 'fixed' = insulator with a bandgap
    smearing = 'mv'                 ! only used when occupations = 'smearing'; type of smearing used
    degauss = NUM
    nbnd = NUM                      ! total number of electronic bands to be modelled
/
&ELECTRONS
  conv_thr = 1.0d-NUM               ! if Smear: increase by ~10^2 from Relaxation; if Fixed: increase by ~ 10^2 from Smear
  mixing_beta = NUM                 ! mixing factor for self-consistency
  mixing_mode = 'plain'             ! type of electron mixing
  mixing_ndim = 16
  electron_maxstep = 1000           ! max number of iterations in SCF cycle
  diagonalization = 'david'         ! type of diagonalization used to process electronic Hamiltonian
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

K_POINTS automatic
    k1 k2 k3  0 0 0
```
### Significant Parameters:

**tot_charge** - this is the total charge of your system. Charges are computed during SCF jobs; no other jobs recalculate this charge, so SCF is the only job that needs this parameter to be specified.

Positive charges are denoted as standard integers (no + sign required). Negative charges are denoted as negative integers (use the - sign). If not specified, tot_charge defaults to 0.

**occupations** - 

More information on occupations can be found in [Smear vs. Fixed Occupations](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear%20vs.%20Fixed%20Occupations.md)

**nbnd** - this is the total number of electronic states (bands) that will be calculated in a job submission. For insulators, the convention is nbnd = # of electrons/2 (or the number of valence bands). For metals, the convention is nbnd = 1.2(# of electrons/2) (or 20% more than the number of valence bands).

nbnd becomes important for future calculations that rely on a range of electron bands, such as DOS, PDOS, and Bands. Because SCF establishes the electronic structure of your material for all future calculations, it is important you choose the correct nbnd now. 

**conv_thr** - this is the convergence threshold for your calculation, determining when the Kohn-Sham matrix is considered to have 'converged'. For convergece to be acheived, *the estimated energy error must be less than than the conv_thr value.*

A lower conv_thr corresponds to a faster calculation with a higher chance of acheiving convergence at a lower accuracy/precision. A higher conv_thr corresponds to a slower calculation with a lower chance of acheiving convergence at a higher accuracy/precision. Higher conv_thr are needed for more advanced calculations, such as phonon and atomic forces.

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

echo "Starting SMEAR: $(date)"     ! written to .out file
export OMP_NUM_THREADS=1           ! MPI threading                 
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} < smear.in >> smear.out   ! run line

echo "Completed SMEAR: $(date)"    ! written to .out file
```

### Significant Parameters:
**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)

## Key Outputs

**HOMO, LUMO, and Bandgap**

**Fermi Level**

**Number of K-Points**
