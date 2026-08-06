# Bands Jobs

### Table of Contents
1. [Bands Job Basics](#bands-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Plotting Band Structures](#plotting-band-structures)


## Bands Job Basics
**Purpose:** to post-process the raw data generated from a NSCF job into a easily graphed format. Reads the raw binary files and connects discrete eigenvalues into continuous bands while performing symmetry analysis.

Must be performed after a NSCF job.

**Package:** bands.x

**Resource/Time Usage:** low

An example [Bands input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Bands%20Jobs/bands.in) is attached for your reference (2x2x2 Tetragonal Barium Titanate Supercell).

## Input File Structure
```
&bands
   prefix = 'NAME'        ! labels and collects data associated with this job, MUST BE THE SAME as directory containing wavefunctions
   outdir = './tmp'       ! where wavefunctions and other data is stored
   filband = 'bands.dat'  ! name of .dat output file
   lsym = .true.          ! classifies bands wrt irreducible representations of k points
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
 
echo "Starting BANDS: $(date)".      ! written to .out file
export OMP_NUM_THREADS=1             ! MPI threading
mpirun -np ${NTASKS} bands.x -npool ${NPOOL} -pd .true. < bands.in >> bands.out     ! run line

echo "Completed BANDS: $(date)"      ! written to .out file
```
### Significant Parameters:
**-pd .true.** - see [NSCF Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/NSCF.md#significant-parameters-1)

**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)

## Plotting Band Structures
Runs from a bands.dat.gnu file (from bands.x)
K-point x-value and Fermi Level are obtained from the scf output (fixed.out)

```
import matplotlib.pyplot as plt
from matplotlib import rcParamsDefault
import numpy as np

plt.rcParams["figure.dpi"]=150
plt.rcParams["figure.facecolor"]="white"
plt.rcParams["figure.figsize"]=(8, 6)

# load data
data = np.loadtxt('path/to/file.dat.gnu' \
'')

E_fermi = NUM

k = np.unique(data[:, 0])
bands = np.reshape(data[:, 1], (-1, len(k)))

bands_shifted = bands - E_fermi

for i in range(bands_shifted.shape[0]):
    plt.plot(k, bands_shifted[i, :], linewidth=1, alpha=0.5, color='k')

plt.xlim(min(k), max(k))
plt.axhline(0.0, linestyle=(0, (5, 5)), linewidth=0.75, color='k', alpha=0.5)
# High symmetry k-points (check bands_pp.out)
plt.axvline(0.0000, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(0.5000, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(1.0000, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(1.7071, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(2.2017, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(2.7017, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(3.2017, linewidth=0.75, color='k', alpha=0.5)
plt.axvline(3.9088, linewidth=0.75, color='k', alpha=0.5)
# text labels
plt.xticks(ticks= [0, 0.5, 1.0, 1.7071, 2.2017, 2.7017, 3.2017, 3.9088], labels=['$\Gamma$', 'X', 'M', '$\Gamma$', 'Z', 'R', 'A', 'Z'])   ! these x values and labels can be changed to match your system geometry
plt.ylabel("$E - E_f$ (eV)")
plt.xlabel("K-path")
plt.title("Band Structure", fontsize='medium')
plt.ylim(-7, 7)
#plt.show()
plt.savefig("path/to/save/file.png", dpi=600)
```
