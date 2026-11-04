# Projected Density of States (PDOS) Jobs

### Table of Contents
1. [PDOS Job Basics](#pdos-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submision-file-structure)
4. [Plotting PDOS](#plotting-pdos)
5. [Plotting Individual Orbitals](#plotting-individual-orbitals)
6. [Example PDOS Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Projected%20Density%20of%20States%20(PDOS)%20Jobs/pdos.in)

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
### Significant Parameters:

**ngauss** - defines the type of Gaussian broadening used.
> ngauss = 0: Simple Gaussian (default)
> 
> ngauss = 1: Methfessel-Paxton of order 1 (higher order is specified by larger positive integers)
> 
> ngauss = -1: Cold-Smearing
> 
> ngauss = -99: Fermi-Dirac function

**degauss** - Gaussian broadening, measured in Ry (NOT eV!). Larger degauss parameter results in increased smoothing; less sharp peaks and a more continuous function. Analogous to the 'accuracy' of the PDOS.

**DeltaE** - Energy 'step' of the PDOS file. This parameter determines at what interval the Energy (y-axis) is sampled/calculated; smaller DeltaE will result in smoother functions and more data points. Analogous to the 'precision' of the PDOS.

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
## Plotting Individual Orbitals

Quantum Espresso has functionalities to sum PDOS contributions (sumpdos.x), but no built-in handle to process individual orbitals within a shell (ie. Px, Py, Pz in P shell). The simplest way to model these specific orbitals is to use the information from a previous PDOS shell file (one of the files produced by the projwfc.x run).

Each PDOS shell file follows this format:

|E (eV) | ldos(E) | pdos(E) | pdos(E) | pdos(E) |
|---|---|---|---|---|
|0.01|0.5|0.3|0.15|0.05|
...

Where the first column is the energy level (y-axis), the second column is the total local density of states (sum of the following columns), and the remaining PDOS columns are individual orbital state contributions. 

In classic QE fashion, the PDOS headers for these files aren't overly useful, as they don't specify which energy corresponds to which orbital in the shell. The solution to this is written in the ylmr2.90 file in QE's code, which gives the order the orbitals are displayed in the shell files:

| Shell | Momentum Component | Plain-English Label | Order in Shell File (L->R)|
|---|---|---|---|
|S|$P_{0,0}(t)$|$S$|1|
|P|$P_{1,0}(t)$|$P_z$|1|
|P|$P_{1,1}cos \phi (t)$|$P_x$|2|
|P|$P_{1,1}sin \phi (t)$|$P_y$|3|
|D|$P_{2,0}(t)$|$D_{z^2}$|1|
|D|$P_{2,1}cos \phi (t)$|$D_{xz}$|2|
|D|$P_{2,1}sin \phi (t)$|$D_{yz}$|3|
|D|$P_{2,2}cos2 \phi (t)$|$D_{x^2  -y^2}$|4|
|D|$P_{2,2}sin2 \phi (t)$|$D_{xy}$|5|

Note: the orbitals are referred to by their momentum component, not plain-english label, in the QE files.

The following script can then be used to parse the orbital energies from the above columns:
```
import os
import matplotlib.pyplot as plt
import numpy as np

######## MANUAL INPUT ##########
Fermi_level = NUM  # eV, from excel sheet
orb_type = 'p'         # 's', 'p', or 'd'
fname = 'path/to/pdos_file'
outdir = 'path/to/save_file'
################################

###### SETS NUMBER OF ORBITALS PER SHELL ######
ORBITALS = {
    's': [('s', 2, r'$s$')],
    'p': [('pz', 2, r'$p_z$'), ('px', 3, r'$p_x$'), ('py', 4, r'$p_y$')],
    'd': [('dz2', 2, r'$d_{z^2}$'), ('dxz', 3, r'$d_{xz}$'),
          ('dyz', 4, r'$d_{yz}$'), ('dx2y2', 5, r'$d_{x^2-y^2}$'),
          ('dxy', 6, r'$d_{xy}$')],
}

if orb_type not in ORBITALS:
    raise ValueError(f"ERROR: no valid orbital type specified! got {orb_type!r}")

data = np.loadtxt(fname)
energy = data[:, 0] - Fermi_level   # shifted: Fermi level -> 0 eV

xmin, xmax = 5 - Fermi_level, 18 - Fermi_level
ymax = data[:, 2:].max() * 1.1

######### PLOT INDIVIDUAL PNGs PER ORBITAL ########
for name, col, label in ORBITALS[orb_type]:
    pdos = data[:, col]

    fig, ax = plt.subplots(figsize=(8, 4))
    ax.plot(energy, pdos, linewidth=0.75, color='#d62728', label=f'{label} orbital')
    ax.axvline(x=0, linewidth=0.5, color='k', linestyle=(0, (8, 10)))
    ax.fill_between(energy, 0, pdos, where=(energy < 0),
                    facecolor='#d62728', alpha=0.25)
    ax.set_xlabel(r'$E - E_f$ (eV)')
    ax.set_ylabel('PDOS (states/eV)')
    ax.set_xlim(xmin, xmax)
    ax.set_ylim(0, 6)
    ax.legend(frameon=False)

    out = os.path.join(outdir, f'pdos_{name}.png')
    fig.savefig(out, dpi=300, bbox_inches='tight')
    plt.close(fig)
    print(f'saved {out}')
```
