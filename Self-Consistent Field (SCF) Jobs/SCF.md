# Self-Consistent Field (SCF) Jobs

### Table of Contents
1. [SCF Job Basics](#scf-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Smear vs. Fixed Occupations](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear%20vs.%20Fixed%20Occupations.md)
5. [Example Smear SCF Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear.in)
6. [Example Fixed SCF Input File](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Fixed.in)


## SCF Job Basics

**Purpose:** To determine the ground-state energy and electron density of a system. This is accomplished by self-consistently solving the Kohn-Sham equation and constructing discrete electron orbitals. ***Essential for more advanced calculation types.***

**Package:** pw.x

**Resource/Time Usage:** medium/high

An example [Smear SCF input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smear.in) and [Fixed SCF input file](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Fixed.in) are attached for your reference (2x2x2 Tetragonal Barium Titanate Supercell).

## Input File Structure
```
&CONTROL
    calculation = 'scf'
    prefix = 'NAME'
    outdir = './tmp'
    pseudo_dir = '/home/kylay/scratch/PAW_BTO/psueds'
    tprnfor = .true.
    tstress = .true.
    etot_conv_thr = 1.0d-4  ! Ry
    forc_conv_thr = 1.0d-3  ! Ry/au
    nstep = 500
    disk_io = 'high'
    restart_mode = 'from_scratch'
/
&SYSTEM
    tot_charge = 2
    ibrav = 0
    nat = 40
    ntyp = 3
    ecutwfc = 60
    ecutrho = 600
    occupations = 'smearing'
    smearing = 'mv'
    degauss = 0.005
    nbnd = 200                    ! total number of electronic bands to be modelled (more for FIXED job)
/
&ELECTRONS
  conv_thr = 1.0d-8               ! increased convergence threshold compared to RELAX, but less than FIXED
  mixing_beta = 0.4               ! best value for BTO
  mixing_mode = 'plain'
  mixing_ndim = 16
  electron_maxstep = 200
  diagonalization = 'david'
/
ATOMIC_SPECIES
    Ba  137.327  Ba.pbe-spn-kjpaw_psl.1.0.0.UPF
    Ti  47.867   Ti.pbe-spn-kjpaw_psl.1.0.0.UPF
    O   15.999   O.pbe-n-kjpaw_psl.1.0.0.UPF

CELL_PARAMETERS angstrom
    7.98400000    0.00000000    0.00000000
    0.00000000    7.98400000    0.00000000
    0.00000000    0.00000000    8.07200000

ATOMIC_POSITIONS crystal
                                 ! use optimized coordinates from RELAX job
K_POINTS automatic
    3 3 3  0 0 0
```
### Significant Parameters:

## Submission File Structure
```
#!/bin/bash
#SBATCH --job-name=NAME
#SBATCH --account=def-ipaci
#SBATCH --nodes=1
#SBATCH --ntasks=192
#SBATCH --cpus-per-task=1
#SBATCH --time=00:30:00
#SBATCH --mem=0
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.err

VERBOSE=TRUE
module --force purge
module load StdEnv/2023
module load quantumespresso/7.5

cd ${SLURM_SUBMIT_DIR}

NTASKS=${SLURM_NTASKS}
NPOOL=8                   

echo "Starting SMEAR: $(date)"
export OMP_NUM_THREADS=1
mpirun -np ${NTASKS} pw.x -npool ${NPOOL} < smear.in >> smear.out

echo "Completed SMEAR: $(date)"
```

### Significant Parameters:

