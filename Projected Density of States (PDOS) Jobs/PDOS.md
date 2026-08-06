# Projected Density of States (PDOS) Jobs

### Table of Contents
1. [PDOS Job Basics](#pdos-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submision-file-structure)
4. [Plotting PDOS](#plotting-pdos)
5. [Example PDOS Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Projected%20Density%20of%20States%20(PDOS)%20Jobs/pdos.in)

## PDOS Job Basics
**Purpose:** to project wavefunctions generated from SCF runs onto atomic orbitals. This allows visualization of individual atomic contributions to the total electron density of the material.

Must be performed after a Fixed SCF job.

**Package:** projwfc.x

**Resource/Time Usage:** low

An example [PDOS input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Projected%20Density%20of%20States%20(PDOS)%20Jobs/pdos.in) is attached for your reference (2x2x2 Tetragonal Barium Titanate).


## Input File Structure

```
&PROJWFC
   prefix = 'NAME'
   outdir = './tmp/'
   filpdos = 'NAME_pdos.dat'
   ngauss = 0
   degauss = 0.01
   DeltaE = 0.005
   Emin = 0 !eV, NOT relative to fermi level (abs values)
   Emax = 20.0 !eV
   lwrite_overlaps = .true.  !gives more accurate projections for PAW pseudos
   lbinary_data = .false.    !produces data in python-readable file
   kresolveddos = .false.    !want atom resolved dos
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

echo "Starting PROJWFC: $(date)"     ! written to .out file
export OMP_NUM_THREADS=1             ! MPI threading
mpirun -np ${NTASKS} projwfc.x -pd .true. < projwfc.in >> projwfc.out      ! run line

echo "Completed PROJWFC: $(date)"   ! written to .out file
```
### Significant Parameters:
**-pd .true.** - see [NSCF Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Non-Self-Consistent%20Field%20(NSCF)%20Jobs/NSCF.md#significant-parameters-1)

**NPOOL** - see [Relaxation Jobs](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Relaxation%20Jobs/Relax.md#significant-parameters-1)


## Plotting PDOS
Single PDOS Plot:
```
import matplotlib.pyplot as plt
from matplotlib import rcParamsDefault
import numpy as np

########### FERMI LEVEL ############
Fermi_level = NUM  #in eV, CHANGE PER RUN

# load data
def data_loader(fname):
    import numpy as np

    data = np.loadtxt(fname)
    energy = data[:, 0]
    pdos = data[:, 1]  # ldos col, total contribution for a given orbital

    return energy, pdos

energy, pdos_s = data_loader('path/to/s_orbital_file')
_, pdos_p = data_loader('path/to/p_orbital_file')
pdos_tot = pdos_s + pdos_p

# make plots
plt.figure(figsize = (8, 4))
plt.plot(energy, pdos_s, linewidth=0.75, color='#006699', label='s-orbital')
plt.plot(energy, pdos_p, linewidth=0.75, color='r', label='p-orbital')
plt.plot(energy, pdos_tot, linewidth=0.75, color='k', label='total')
plt.yticks([])
plt.xlabel('Energy (eV)')
plt.ylabel('DOS')
plt.axvline(x= Fermi_level, linewidth=0.5, color='k', linestyle=(0, (8, 10)))
plt.xlim(5, 18)
plt.ylim(0, )
plt.fill_between(energy, 0, pdos_s, where=(energy < Fermi_level), facecolor='#006699', alpha=0.25)
plt.fill_between(energy, 0, pdos_p, where=(energy < Fermi_level), facecolor='r', alpha=0.25)
plt.fill_between(energy, 0, pdos_tot, where=(energy < Fermi_level), facecolor='k', alpha=0.25)
# plt.text(6.5, 0.52, 'Fermi energy', fontsize= small, rotation=90)
plt.legend(frameon=False)
plt.show()
```

Array PDOS Plots:
```
import matplotlib.pyplot as plt
from matplotlib import rcParamsDefault
import numpy as np
import glob
import re
import os
from collections import defaultdict

####### MANUAL INPUT #########
Fermi_level = NUM   #in eV
name = 'NAME'  #must be the same as the extracted zip folder ("NAME_pdos.zip")
##############################

###### SCEONDARY VARIABLES ########
data_dir = 'path/to/zip/folder/%s' % name
save_dir = 'path/to/save/folder/%s/' % name  #set to 'None' to skip saving
##################################

ORBITAL_COLOURS = {
   's' : '#44ACEB',
   'p' : '#EB4444',
   'd' : '#A218E7',
}

def data_loader(fname):
            data = np.loadtxt(fname)
            energy = data[:, 0]
            pdos = data[:, 1] 
            return energy, pdos

######### PULL NAME VARIABLES ###########
pattern = re.compile(r'atm[#_](\d+)[_\(]([A-Za-z]+)\)?_*wfc[#_](\d+)[_\(]([a-z]+)\)?_?')

all_entries = sorted(glob.glob(os.path.join(data_dir, '*')))
all_files = [f for f in all_entries if pattern.search(os.path.basename(f))]

if not all_files:
   print('Looked in: %s' % data_dir)
   print('Found %d entries there. First 10:' % len(all_entries))
   for f in all_entries[:10]:
      print('  ', os.path.basename(f))
   raise FileNotFoundError('No PDOS files found in %s' % data_dir)

######## SORT FILES BY ATOM NUM #######
atoms = defaultdict(list) #atom_num -> list of (element, wfc_id, orbital, fname)
for f in all_files:
   match = pattern.search(os.path.basename(f))
   if not match:
      continue
   atom_num, element, wfc_id, orbital = match.groups()
   atoms[atom_num].append((element, wfc_id, orbital, f))

######## LOOP OVER ATOMS ###########
for atom_num, entries in sorted(atoms.items(), key=lambda x: int(x[0])):
   element = entries[0][0]  #element is the same for all entires of this atom

   energy = None
   orbital_data = {}  #orbital type -> pdos array (summed if multiple wfc of same type)

   for elem, wfc_id, orbital, fname in entries:
      e, pdos = data_loader(fname)
      if energy is None:
         energy = e
      if orbital in orbital_data:
         orbital_data[orbital] = orbital_data[orbital] + pdos
      else:
         orbital_data[orbital] = pdos

   pdos_tot = sum(orbital_data.values())

   ####### PLOT (PER ATOM) #########
   plt.figure(figsize = (8, 4))
   for orbital, pdos in sorted(orbital_data.items()):
       color = ORBITAL_COLOURS.get(orbital, 'gray')
       plt.plot(energy, pdos, linewidth=0.75, color=color, label='%s-orbital' % orbital)
       plt.fill_between(energy, 0, pdos, where=(energy < Fermi_level), facecolor=color, alpha=0.25)

   plt.plot(energy, pdos_tot, linewidth=0.75, color='k', label='total')    
   plt.fill_between(energy, 0, pdos_tot, where=(energy < Fermi_level), facecolor='k', alpha=0.25)
  
   plt.yticks([])
   plt.xlabel('Energy (eV)')
   plt.ylabel('DOS')
   plt.axvline(x= Fermi_level, linewidth=0.5, color='k', linestyle=(0, (8, 10)))
   plt.xlim(5, 18)
   plt.ylim(0, )
   plt.title('PDOS: %s, Atom #%s (%s)' % (name, atom_num, element))
   plt.legend(frameon=False)

   if save_dir:
      os.makedirs(save_dir, exist_ok=True)
      plt.savefig(os.path.join(save_dir, '%s_atom%s_%s' % (name, atom_num, element)))
```
