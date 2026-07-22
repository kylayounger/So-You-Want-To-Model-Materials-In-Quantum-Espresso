# Why Fractional Coordinates?

## What are Fractional Coordinates?
Fractional coordinates list the positions of atoms as fractions of the lattice vectors, and are always decimal values between 0-1. 

$$\vec{r} = x\vec{a} + y\vec{b} + z\vec{c}$$

Where: $\vec{r}$ are the Cartesian coordinates, $x, y, z$ are the Fractional coordinates, and $\vec{a}, \vec{b}, \vec{c}$ are the direct lattice basis vectors.

## Why does Quantum Espresso process Fractional Coordinates differently?

When Quantum Espresso reads Cartesian coordinates, it converts them to Fractional form using the following equation:

$$\vec{r_{frac}} = A^{-1} \cdot \vec{r_{cart}}$$

Where: $\vec{r_{frac}}$ is a Fractional coordinate, $\vec{r_{cart}}$ is a Cartesian coordinate, and $A^{-1}$ is the inverse of the lattice matrix.

This introduces a source of error: 

However, when Quantum Espresso reads Fractional coordinates the program doesn't need to perform a conversion - the positions are already in the internal representation that the code uses for matrix operations. This means that the error propogation that occurs during Cartesion-to-Fractional conversion is not present.
