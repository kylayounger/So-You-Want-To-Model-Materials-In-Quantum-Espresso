# Combining Jobs (Pipeline)

### Table of Contents
1. [Pipeline Basics](#pipeline-basics)
2. [Pipeline Submission Script](#pipeline-submission-script)


## Pipeline Basics
**Purpose:** to connect seperate, modular jobs into a single submission.

**Package:** custom; whichever packages your individual jobs use.

**Resource/Time Usage:** High

## Pipeline Submission Script
```
#!/bin/bash
#SBATCH --job-name=NAME              ! this name will appear in the queue
#SBATCH --account=ACCT               ! links to resource allocation
#SBATCH --nodes=1                    ! number of computer nodes requested
#SBATCH --ntasks=192                 ! number of CPUs in requested node
#SBATCH --cpus-per-task=1 
#SBATCH --time=03:59:00              ! IMPORTANT
#SBATCH --mem=0                      ! memory allocation, 0 = unlimited
#SBATCH --output=%x_%j.out           ! writes an additional output file
#SBATCH --error=%x_%j.err            ! writes an error file

VERBOSE=TRUE                         ! ensures output files are as detailed as possible
module --force purge                 ! clears previous modules and loads necessary ones
module load StdEnv/2023
module load quantumespresso/7.5

###################### CHECKING FUNCTIONS #####################
check_done() {
    local outfile=$1
    local jobname=$2
    if grep -q "JOB DONE" "${outfile}"; then
        echo "${jobname} completed successfully: $(date)"
    else
        echo "ERROR: ${jobname} did not complete — 'JOB DONE' not found in ${outfile}"
        echo "Stopping pipeline at: $(date)"
        exit 1
    fi
}

print_done() {
    local outfile=$1
    local jobname=$2
    if grep -q "JOB DONE" "${outfile}"; then
       echo "${jobname} completed successfully: $(date)"
    else
       echo "ERROR: ${jobname} did not complete (non-critical failure)"
       exit 1
    fi
}

##################### PIPELINE ##########################

cd ${SLURM_SUBMIT_DIR}               ! creates tmp/ directory
mkdir -p tmp

NTASKS=${SLURM_NTASKS}
NPOOL=NUM                            ! how tasks are divided between nodes/CPUs

export OMP_NUM_THREADS=1             ! environmental variables
export MKL_CBWR=COMPATIBLE
export MKL_DEBUG_CPU_TYPE=5
ulimit -s unlimited                

echo "Starting SMEAR: $(date)"
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} < smear.in >> smear.out
check_done "smear.out" "SMEAR"

echo "Starting FIXED: $(date)"
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} < fixed.in >> fixed.out
check_done "fixed.out" "FIXED"

echo "Starting DOS: $(date)"
mpirun -np ${NTASKS} dos.x -npool ${NPOOL} -pd .true.  < dos.in >> dos.out
print_done "dos.out" "DOS"

echo "Starting PROJWFC: $(date)"
mpirun -np ${NTASKS} projwfc.x -pd .true. < projwfc.in >> projwfc.out
print_done "projwfc.out" "PROJWFC"

echo "Starting NSCF: $(date)"
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} -pd .true. < nscf.in >> nscf.out
check_done "nscf.out" "NSCF"

echo "Starting BANDS: $(date)"
mpirun -np ${NTASKS} bands.x -npool ${NPOOL} -pd .true. < bands.in >> bands.out
print_done "bands.out" "BANDS"

echo "Starting PHONON: $(date)"
mpirun -np ${NTASKS} ph.x -npool ${NPOOL} -pd .true. < ph.in >> ph.out
check_done "ph.out" "PHONON"

echo "Starting DYNMAT: $(date)"
mpirun -np ${NTASKS} dynmat.x -npool ${NPOOL} -pd .true. < dynmat.in >> dynmat.out
print_done "dynmat.out" "DYNMAT"

echo "Pipeline Concluded Successfully: $(date)"         !  written to output file
```
