# Relaxation Jobs
**Purpose:** To verify and stabilize atomic geometries. Generally conducted before calculating the electronic structure of a given geometry to ensure it is physically reasonable.

**Package:** pw.x

**Resource/Time Usage:** low/medium

An [example relaxation input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.in) is attached for your reference (2x2x2 Barium Titanate Supercell).

## Input File Structure:
```
&CONTROL
  calculation = 'relax'             ! designates RELAX as the type of job to run
  prefix = 'NAME'                   ! labels and collects data associated with this job
  outdir = './tmp'                  ! where wavefunctions and other data is stored (directory must be created before running job)
  pseudo_dir ='/path/to/pseuds'
  tstress = .true.
  tprnfor = .true.
  etot_conv_thr = 1.0d-4  ! Ry
  forc_conv_thr = 1.0d-3  ! Ry/au
  nstep = 500                       ! max number of optimization steps allowed in job
  restart_mode = 'from_scratch'     !'from_scratch' = completly restarts, 'restart' = continue from where last job finished
/
&SYSTEM
  ibrav = 0                         ! cell type; 0 = free cell (determined by CELL_PARAMETERS)
  nat = NUM                         ! total number of atoms in your base cell/supercell
  ntyp = NUM                        ! total number of elements in your system
  ecutwfc = NUM                     ! kinetic energy cutoff for wfcs
  ecutrho = NUM                     ! kinetic energy cutoff for charge density, 4*ecutwfc is default
  occupations = 'smearing'          ! 'smearing' = smeared electrons for metals, 'fixed' = insulator with a bandgap (smearing converges faster, but fixed is needed for Ph.x)
  smearing = 'mv'                   ! type of smearing used
  degauss = NUM                     ! gaussian spreading in Brillouin zone integration
/
&ELECTRONS
  conv_thr = 1.0d-NUM               ! conv_thr can be low here as tight convergence isn't needed for structural verification
  mixing_beta = NUM                 ! mixing factor for self-consistency
  mixing_mode = 'plain'             ! type of electron mixing
  mixing_ndim = 16
  electron_maxstep = 1000           ! max number of iterations in SCF cycle
  diagonalization = 'david'         ! type of diagonalization used to process electronic Hamiltonian
/
&IONS
/
ATOMIC_SPECIES
    ATOM  ATOMIC_MASS  FILENAME     ! links to your pseudopotential files 
    ATOM  ATOMIC_MASS  FILENAME     ! all atom types in your cell must have a pseudopotential file listed here

CELL_PARAMETERS angstrom
    x    0.0    0.0                 ! dimensions of your unit cell/supercell (only needed for ibrav = 0)
    0.0    y    0.0
    0.0    0.0    z

ATOMIC_POSITIONS crystal
! Insert FRACTIONAL atomic positions here

K_POINTS automatic
   k1 k2 k3 0 0 0
```
### Significant Parameters:
**ibrav** - this is the Bravais Lattice index of your unit cell (also called the crystal space group). ibrav inputs are integers from 0-14, each denoting a different Bravais Lattice type. By selecting one of these options, Quantum Espresso will use pre-set Bravais Lattice constants to model your geometry:
> ibrav = 0, free geometry, lattice specified in CELL_PARAMETERS
> 
> ibrav = 1, Cubic P (sc)
> 
> ibrav = 2, Cubic F (fcc)
> 
> ...
> 
> ibrav = 14, Triclinic

**CELL_PARAMETERS** - for free geometry cells (ibrav = 0), you must specify the lattice parameters you wish to implement. This is done in the CELL_PARAMETERS block, and are generally sourced from experimental values. It should be noted that CELL_PARAMETERS account for the *total* lattice parameter if your supercell, *not* the lattice parameter of your unit cell. 

For example, for a 2x2x2 Tetragonal Barium Titanate Cell, I would list the unit cell parameters (a, b, c) x2:
> CELL_PARAMETERS angstrom
> 
> 2a 0.0 0.0
> 
> 0.0 2b 0.0
> 
> 0.0 0.0 2c

**ATOMIC_POSITIONS** - the .xyz geometry you wish to optimize. These are generally made in avogadro (or similar builder programs) and imported to the input file. The unit of these positions can be changed using the modifier next to the ATOMIC_POSITIONS header.

For best results, ***your atomic positions MUST be in fractional coordinates!!*** This is because Quantum Espresso . This is especially important for systems with low internal symmetry (such as tetragonal unit cells, vacancies, etc) as Quantum Espresso's less-robust cartesian processing will likely lead to job failure or unreasonable geometries.

## Submission File Structure:
```
#!/bin/bash
#SBATCH --job-name=NAME             ! this name will appear in the queue
#SBATCH --account=ACCT              ! links to resource allocation
#SBATCH --nodes=1                   ! number of computer nodes requested
#SBATCH --ntasks=192                ! number of CPUs in requested node
#SBATCH --cpus-per-task=1
#SBATCH --time=23:00:00             ! IMPORTANT
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

echo "Starting RELAX: $(date)"     ! written to .out file
export OMP_NUM_THREADS=1           ! MPI threading
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} < NAME_relax.in >> NAME_relax.out     ! run line

echo "Completed RELAX: $(date)"    ! written to .out file
```
### Significant Parameters:
**NPOOL** - dictates how the k-point grid is divided into independant processing pools. This is an important part of the parallelization of your job, and can greatly increase/decrease the efficiency of your job submission. To find the correct NPOOL integer for your geometry, you factor the number of irreducible k-points in your calculation to find an appropraite division of tasks. 

For example, if your system has 40 irreducible k-points, possible NPOOL numbers are 1, 2, 4, 5, 8, 10, 20.

The number of irreducible k-points can be found by searching 'number of k points' in the relax.out file; unfortunatly, this means you have to guess NPOOL upon your first job submission. When in doubt, NPOOL = 1 will always run successfully, but may be time consuming.
