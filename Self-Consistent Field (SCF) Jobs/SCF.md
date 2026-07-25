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

### Significant Parameters:

## Submission File Structure

### Significant Parameters:

