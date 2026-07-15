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

**2. Complex Calculations:**



## Useful Resources
These resources have been invaluable in my journey in learning Quantum Espresso! Much of the information I relay here is sourced from the following:

### Quantum Espresso Documentation:
https://www.quantum-espresso.org/documentation/

### Density Functional Theory Using Quantum Espresso (blog):
https://pranabdas.github.io/espresso/

### Materials Square (blog):
https://www.materialssquare.com/


