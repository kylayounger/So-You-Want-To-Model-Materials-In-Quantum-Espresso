# Post Processing (PP) Jobs
### Table of Contents
1. [PP Job Basics](#pp-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submision-file-structure)
4. [Plotting PP](#plotting-pp)
5. [Example PP Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Post%20Processing%20(PP)%20Jobs/pp.in)

## PP Job Basics
**Purpose:**

**Package:** pp.x

**Resource/Time Usage:** low

An example [PP input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Post%20Processing%20(PP)%20Jobs/pp.in) is attached for your reference (2x2x2 Tetragonal Barium Titanate).

## Input File Structure
```
&INPUTPP
   prefix = 'NAME'               ! must be same as directory handle
   outdir = './tmp/'             ! links to SCF wavefunctions
   plot_num = 11                 ! outputs vacuum potential in Rydberg (V_bare + V_H potential)
   filplot = 'total_pot.dat'     ! filename of file that contains quantity selected by plot_num
/
&PLOT
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

echo "Starting PP: $(date)"       ! written to .out file
export OMP_NUM_THREADS=1           ! MPI threading
mpirun -np ${NTASKS} pp.x -npool ${NPOOL} -pd .true. < pp.in >> pp.out      ! run line

echo "Completed PP: $(date)"      ! written to .out file
```
