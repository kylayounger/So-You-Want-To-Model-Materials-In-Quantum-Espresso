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

## Modelling Fractional Coordinates: VESTA  and Python

By default, most molecular modelling programs will write atomic coordinates in XYZ cartesian coordinates. This is easy enough to convert to fractional coordinates using a Python script - however, there are a few quirks you should know about generating your cartesian geometry before converting.

**Cartesian to Fractional Coordinates Conversion Script:**
```
import numpy as np
# Must change CELL PARAMETERS to match the lattice parameters of the cell/supercell!

def convert_xyz_to_fractional(xyz_filename, lattice_vectors, output_filename):
    M = np.array(lattice_vectors).T
    try:
        M_inv = np.linalg.inv(M)
    except np.linalg.LinAlgError:
        raise ValueError("Lattice vectors are coplanar or invalid; matrix is singular.")

    with open(xyz_filename, 'r') as f:
        lines = f.readlines()

    try:
        num_atoms = int(lines[0].strip())
    except ValueError:
        raise ValueError("Invalid .xyz format: Line 1 must state the number of atoms.")

    comment = lines[1].rstrip('\n')
    atom_data = lines[2:2 + num_atoms]

    fractional_atoms = []

    for line in atom_data:
        parts = line.split()
        if len(parts) < 4:
            continue

        element = parts[0]
        cartesian_coords = np.array([float(parts[1]), float(parts[2]), float(parts[3])])
        fractional_coords = np.dot(M_inv, cartesian_coords)
        fractional_atoms.append((element, *fractional_coords))

    with open(output_filename, 'w') as f:
        f.write(f"{len(fractional_atoms)}\n")
        f.write(f"Fractional coordinates converted from {xyz_filename}\n")
        for element, fx, fy, fz in fractional_atoms:
            f.write(f"{element:2s} {fx:.8f} {fy:.8f} {fz:.8f}\n")

#CELL PARAMETERS
if __name__ == "__main__":
    unit_cell = [
        [ NUM,  0.000,  0.000],  # Vector a
        [ 0.000,  NUM,  0.000],  # Vector b
        [ 0.000,  0.000,  NUM]   # Vector c
    ]

    # Run conversion
    convert_xyz_to_fractional("path/to/input/file", unit_cell, "path/to/output/file")
```
