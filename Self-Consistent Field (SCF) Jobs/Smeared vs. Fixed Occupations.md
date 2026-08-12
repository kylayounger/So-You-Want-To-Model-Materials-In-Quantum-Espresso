# Smeared vs. Fixed Occupations

### Table of Contents
1. [What are Occupations?](#what-are-occupations)
2. [Fixed Occupations](#fixed-occupations)
3. [Smeared Occupations](#smeared-occupations)
4. [Additional Resources](#additional-resources)

## What are Occupations?

## Fixed Occupations

When occupations = 'fixed', electron occupations are modelled using the **Heaviside Step Function:**

```math
u_{nk} = 1 \; if \;\; E_{nk} < E_f \; ; \;\;\;\;
u_{nk} = 0 \; if \;\; E_{nk} > E_f
```
<br>

Where $u_{nk}$ is the occupation number at a certain band index (n) and k-point (k), $E_{nk}$ is the energy (or Kohn-Sham eigenvalue) at a given band index (n) and k-point (k), and $E_f$ is the energy of the Fermi level.

Simply put, this step function states that all electronic orbitals with energy less than the Fermi level will be occupied, and that all orbitals with energy greater than the Fermi level will be unoccupied. This function is **discontinuous**; $u_{nk}$ can only ever equal 0 or 1 at a given energy level. Because of this inherit discontinuity, fixed occupations **must have a discrete occupancy value** - this means that Quantum Espresso must be able to determine if each orbital within your system is $u_{nk} = 0$ or $u_{nk} = 1$ without ambiguity.

For systems with a well-established band gap (ie. insulators), the HOMO and LUMO orbitals are clearly defined and the Fermi level sits well within the band gap. These are the systems for which Quantum Espresso recommends using occupations = 'fixed'; this setting is generally not recommended for metals and semi-conductors. 

This effect can also be seen when defect-containing systems are run with 'fixed' occupations: if a defect state is too close to the Fermi level, it creates a discontinuity where The SCF violently oscillates between $u_{nk} = 0$ and $u_{nk} = 1$. As Quantum Espresso is unable to determine the exact occupancy of this state, the job will fail with 'IEEE denormal' or 'c_bands' errors.

However, in order to run higher-level calculations (such a phonon and dynamical matrix jobs), **electron occupations must be 'fixed'**. Herein lies the issue: if you're modelling a semi-conductor or defect-containing material, **you must still get wavefunctions with 'fixed' occupations**, even though setting occupations = 'fixed' results in a discontinuity and causes the job to fail.

**So...how do we get around this?**

## Smeared Occupations

Disclaimer: There are multiple types of smearing within Quantum Espresso; I will be discussing Methfessel-Vanderbilt (MV) smearing, as it is what I’m the most familiar with using.

When occupations = ‘smearing’ and smearing = ‘mv’, electron occupations are modelled using a Fermi-Dirac Smoothing function $f(x)$ :

```math
f(x) =  \frac{1}{2} \; erfc(x) + \; \frac{1}{\sqrt{\pi}} \Sigma_{n=1}^{N} \; A_n \; H_{2n+1}(x) \; e^{-x^2}
```
<br>
Where $erfc(x)$ is the complimentary error function, $A_n$ are expansion coefficients, and $H_n$ are Hermite polynomials.

The value of $x$ is determined by:

```math
x = \left( E_{nk} - E_f \right) / \sigma
```
<br>
Where $E_{nk}$ is the energy (or Kohn-Sham eigenvalue) at a given band index (n) and k-point (k), $E_f$ is the energy of the Fermi level, and $\sigma$ is the degauss value.


## Sources and Resources

[N. Marzari, D. Vanderbilt, A. De Vita, and M. C. Payne, "Thermal Contraction and Disordering of the Al(110) Surface", Physical Review Letters 82, 3296 (1999).](https://doi.org/10.1103/PhysRevLett.82.3296)

[M. Methfessel and A. T. Paxton, Physical Review B 40, 3616 (1989)](https://doi.org/10.1103/PhysRevB.40.3616)

[W. Kohn and L. J. Sham, "Self-consistent equations including exchange and correlation effects", Physical Review 140, A1133 (1965)](https://doi.org/10.1103/PhysRev.140.A1133)

R. M. Martin, "Electronic Structure: Basic Theory and Practical Methods" (Cambridge University Press, 2004). eISBN 9781108555586.
