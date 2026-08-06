# Density of States (DOS) Jobs

### Table of Contents
1. [DOS Job Basics](#dos-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Plotting DOS](#plotting-dos)

## DOS Job Basics
**Purpose:** to post-process occupation and orbital information generated from SCF runs into easily graphed Density of States format. Evaluates the number of electronic states available at each energy level.

Must be performed after a Smeared/Fixed SCF job.

**Package:** dos.x

**Resource/Time Usage:** low

An example [DOS input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Density%20of%20States%20(DOS)%20Jobs/dos.in) is attached for your reference (2x2x2 Tetragonal Barium Titanate Supercell).

## Input File Structure
```
&DOS
  prefix = 'NAME'            ! labels and collects data associated with this job, MUST BE THE SAME as directory containing wavefunctions
  outdir = './tmp'           ! where wavefunctions and other data is stored
  fildos = 'dos_gamma.dat'   ! name of the .dat output file

  Emin = -30.0               ! minimum energy value plotted (eV), NOT relative to Fermi Level
  Emax = 30.0                ! maximum energy value plotted (eV), NOT relative to Fermi Level
  DeltaE = 0.005             ! energy grid step; lower = more precise DOS, higher = smoother lines

  ngauss = 0                 ! type of Gaussian broadening; 0 = simple broadening
  degauss = 0.005            ! Gaussian broadening (Ry, not eV)
/
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

echo "Starting DOS: $(date)"       ! written to .out file
export OMP_NUM_THREADS=1           ! MPI threading
mpirun -np ${NTASKS} dos.x -npool ${NPOOL} -pd .true. < dos.in >> dos.out      ! run line

echo "Completed DOS: $(date)"      ! written to .out file
```
### Significant Parameters:
**-pd .true.** - see [NSCF Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/NSCF.md#significant-parameters-1)

**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)

## Plotting DOS

Runs from a dos_gamma.dat file
Need E_Fermi from scf output (fixed.out)

```
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
 
###################### IMPORT DATA ##########################
data = np.loadtxt('path/to/file.dat', comments='#')
energy   = data[:, 0]          # eV (absolute)
dos      = data[:, 1]          # states/eV
int_dos  = data[:, 2]          # integrated DOS
 
####### FERMI LEVEL #######
E_fermi  =  11.2078 # eV CHANGE THIS PER FILE
 
energy_shifted = energy - E_fermi
 
######################### PLOT ##########################
fig, ax = plt.subplots(figsize=(8, 5))
 
# DOS curve
ax.plot(energy_shifted, dos, color='steelblue', linewidth=0.7, solid_joinstyle='miter', solid_capstyle='butt', antialiased=False, label='DOS')
 
# Fill valence band (below Fermi level)
ax.fill_between(energy_shifted, dos,
                where=(energy_shifted <= 0),
                color='steelblue', alpha=0.3, label='Occupied states')
 
# Fill conduction band (above Fermi level)
ax.fill_between(energy_shifted, dos,
               where=(energy_shifted > 0),
              color='tomato', alpha=0.3, label='Unoccupied states')
 
# Fermi level line
ax.axvline(x=0, color='black', linestyle='--', linewidth=1.0, label=r'$E_{\mathrm{F}}$')
 
###################### FORMAT PLOT #######################
ax.set_xlabel('Energy (eV)', fontsize=13)
ax.set_ylabel('Total DOS', fontsize=13)
ax.set_title('TITLE', fontsize=13)
ax.set_xlim(-7.5, 7.5)
ax.set_ylim(bottom=0, top=100)
ax.xaxis.set_minor_locator(ticker.AutoMinorLocator())
ax.yaxis.set_minor_locator(ticker.AutoMinorLocator())
ax.grid(which='major', linestyle=':', linewidth=0.5, alpha=0.7)
 
ax.legend(fontsize=11, framealpha=0.9)
plt.tight_layout()
 
plt.savefig('path/to/save/file.png', dpi=600, bbox_inches='tight')
#plt.show()
```
