# Why Fractional Coordinates?

## What are Fractional Coordinates?

```
$$\vec{r} = x\vec{a} + y\vec{b} + z\vec{c}$$
```

Where: $\vec{r}$ are the Cartesian coordinates, $x, y, z$ are the Fractional coordinates, and $\vec{a}, \vec{b}, \vec{c}$ are the direct lattice basis vectors.

## Why does Quantum Espresso process Fractional Coordinates differently?

```
$$\vec{r_{frac}} = A^{-1} \cdot \vec{r_{cart}}$$
```

Where: $\vec{r_{frac}}$ is a Fractional coordinate, $\vec{r_{cart}}$ is a Cartesian coordinate, and $A^{-1}$ is the inverse of the lattice matrix.
