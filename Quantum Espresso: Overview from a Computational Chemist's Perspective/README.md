# Quantum Espresso: Overview from a Computational Chemist's Perspective
Quantum Espresso is a juggernaut of computational power, but navigating the program's workflow can be challenging for a beginner. Quantum Espresso is an open-source Density Functional Theory (DFT) code that specializes in materials modelling; specifically, by using Plane-Wave basis sets and Pseudopotentials to simplify the quantum states of crystal lattices.

Quantum Espresso is very versatile and hosts a wide range of packages - however, the lack of clear documentation and unclear pathways between these packages can make it a daunting program to learn. Don't get me wrong - **I love Quantum Espresso as much as the next computational chemist** - but there is certainly a learning curve that can be unfriendly to those without a background/degree in computer science (such as myself).

However, Quantum Espresso can be extrordinarily useful once you learn how to use it. In my experince, the most critical point of understanding is the pathway between internal packages; Quantum Espresso is a very modular program, with many different packages that perform many different tasks; this means that a seemingly simple job may require the submission of multiple packages (jobs) to acheive your end goal.

For example, if you want to obtain the band structure of a GaAs unit cell, you would need to run the following job pipeline:
> 1. Relax
> 2. Self-Consistent-Field (SCF)
> 3. Non-Self-Consistent-Field (NSCF)
> 4. Bands

Likewise, calculations aiming to capture more complex elements (such as opto-electronic coefficients) may require more module jobs than you expect, and as such may take more time than you initailly bargained for.

### All that being said, Quantum Espresso has some undeniable advantages over other computational chemistry codes:
**1. Cost:**
Quantum Espresso is *100% free to use and open-source*. This means you're free to download and modify the base code as you like; a major benefit in comparison to similar paid computational tools. If you *are* someone with a computer science background, this makes the program much easier to modify if you wish to make specific changes in elements such as pseudopotentials or custum plug-ins.

**2. Complex Calculations:**
Due to the wide range of internal packages, Quantum Espresso has the capability to perform complex calculations if you link these packages together correctly. Some of these complex packages include:
> 1. Phonon: calculates the acoustic and optical phonon modes using Density Functional Perturbation Theory (DFPT). Used to calculate electro-optic coefficients such as Born Effective Charges, Dielectric Tensors, and  Phonon Modes.
> 2. Nudged Elastic Band (NEB): calculates the transition states (including the Minimum Enegery Path and Activation Energy) of a reaction. Uses intermediate structures/spring forces to model the potential energy surface of a given reaction (currently not included in this repository).
> 3. CP: inlcudes Car-Parinello Molecular Dynamics (CPMD) and Born-Oppenheimer Molecular Dynamics (BOMD) functionalities. This package allows the silmulation of atomic movement, disordered materials, and electron/ion dynamics at both small and large time-scales (currently not included in this repository).

**3. Hardware Scalability:**
Quantum Espresso is easy to use on High Performance Computers (HPCs); the program is designed to distribute workloads over mutiple CPUs/GPUs, making it (relativly) efficient at most calculation types.

**4. Pseudopotential Range:**
Quantum Espresso supports a wide range of psuedopotentials, allowing you to select the type of core-electron approximation that suits your system best. The program supports Projector Augmented Wave (PAW), Ultrasoft, and Norm-Conserving (NC) pseudopotentials.


## Useful Resources
These resources have been invaluable in my journey in learning Quantum Espresso! Much of the information I relay here is sourced from the following:

**Quantum Espresso Documentation:**
https://www.quantum-espresso.org/documentation/

**Density Functional Theory Using Quantum Espresso (blog):**
https://pranabdas.github.io/espresso/

**Materials Square (blog):**
https://www.materialssquare.com/


