# Why Fractional Coordinates?

## What are Fractional Coordinates?
Fractional coordinates list the positions of atoms as fractions of the lattice vectors, and are always decimal values between 0-1. 

$$\vec{r} = x\vec{a} + y\vec{b} + z\vec{c}$$

Where: $\vec{r}$ are the cartesian coordinates, $x, y, z$ are the fractional coordinates, and $\vec{a}, \vec{b}, \vec{c}$ are the direct lattice basis vectors.

Demonstrated another way:
```
    Ba 0.000 0.000 0.000     ! a(0.000), b(0.000), c(0.000)
    Ti 0.250 0.250 0.250     ! a(0.250), b(0.250), c(0.250)
```
Fractional coordinates provide distinct advantages in symmetry representations and non-orthogonal handling. However, fractional coordinates are more difficult to interpret at a glance so are less frequently used.

## Why does Quantum Espresso process Fractional Coordinates differently?

When Quantum Espresso reads cartesian coordinates, it converts them to fractional form using the following equation:

$$\vec{r_{frac}} = A^{-1} \cdot \vec{r_{cart}}$$

Where: $\vec{r_{frac}}$ is a fractional coordinate, $\vec{r_{cart}}$ is a cartesian coordinate, and $A^{-1}$ is the inverse of the lattice matrix.

However, in cartesian coordinates, if an atom falls within a unit vs adjacent cell is determined by the the lattice vectors (a, b, c) and the cell angles. This means that for non-cubic cells, Quantum Espresso must perform a matrix operation to decide what cell an atom belongs to - which can lead to roundings errors and small shifts in atomic positions.

This conversion from cartesian coordinates to fractional coordinates affects some key aspects of Relaxation calculations, especially if you're working with non-cubic cell geometries:

**Periodic Boundary Conditions**

For an atomic position to be considered within the unit cell of a Relaxation calculation, it must fall within the fractional range of [0, 1). Positions with values outside of this range (both positive and negative) are designated to adjacent cells. When writing your atomic positions in fractional coordinates, any values outside this [0, 1) range are obvious, both to you and the code, allowing it to be corrected (either by manually modifying the positions or Quantum Espresso modifying them itself). 

**Orthonormality Issues**

In the 'ideal' case of a cubic cell, the lattice parameters obey the simple rule a = b = c (cubic). However, when working with non-cubic cell types, this relationship is broken. For example, tetragonal cells obey the relationship a = b ≠ c.

When Quantum Espresso converts these tetragonal cartesian coodinates using $A^{-1}$, atomic positions will be affected differently if they are scaled using a,b (the same value) or scaled using c (a slightly larger value). This is commonly seen in rounding errors; if a cartesian coordinate is rounded to fewer decimal places, this error will scale differently on the x/y vs. z planes because the 1/a and 1/c scaling factors are different.

This problem doesn't occur in fractional coordinates, as the positions are exact and require no transformation.

**Symmetry Detection**

*Sym_base.f90* is the source file within Quantum Espresso that is responsible for identifying crystal structures and perofrming symmetry operations, including Brillouin zone k-point generation and real/reciprocal space rotation matrices. *Sym_base.f90* works in fractional coordinates, and checks for translational invariance using the following equation:

$$ S \cdot \tau_i = \tau_j + n $$

Where S is a given symmetry operation, $\tau$ is given fractional coordinate, and n is an integer vector. Essentially, this equation checks if applying a symmetry operation S to fractional coordinate $\tau$ will reult in a translation by integer vector n. 

This vector n defines the spacing between unit cells; it specifies the offset in position between a position in cell 0 and cell 1, and must be exact. However, converting from cartesian to fractional coordinates using $A^{-1}$ can result in small floating-point errors, leading to a slightly different value of n being produced by *sym_base.f90*. This can result in *sym_base.f90* identifying fewer symmetry operations than actually exist, potentially leading to a break in 4-fold symmetry around the c-axis.

**FTT Grid**

The Fast Fourier Transform (FTT) grid is responsible for converting wavefunctions/charge densities between real space and reciprocal space. 

$$ \rho(r) = (1/\Omega) \sum_G \ \rho(G) \ e^{iG \cdot r} $$


## Modelling Fractional Coordinates: VESTA  and Python

By default, most molecular modelling programs will write atomic coordinates in XYZ cartesian coordinates. This is easy enough to convert to fractional coordinates using a Python script - however, there are a few quirks you should know about generating your cartesian geometry before converting.

**Building from a .cif file in VESTA:**

.cif (Crystallographic Information) files contain essential information for modelling a specific crystal type. This information includes symmetry class, unit cell/lattice parameters, and atomic positions of the unit cell. .cif files can be obtained from online databases such as [Materials Project](https://next-gen.materialsproject.org/) or [Crystallography Open Database](https://www.crystallography.net/cod/).

> 1. Open your chosen .cif file in VESTA
> 2. Go to Edit > Edit Bonds
>    
>    a. For each bond type listed, click it and select 'Do not search atoms beyond the boundary' under 'Boundary Mode'; this step allows better   visualization of which atoms are actually being modelled
> 
>    b. Click 'Apply', then 'Okay'
> 
> 3. Go to Edit > Edit Data > Unit Cell...
>    
>    a. Check that the selected symmetry group listed under 'System' matches the crystal you are modelling; if not, change this to match
> 
>    b. Check that the lattice parameters match the the unit cell lengths of the crystal you are modelling; if not, change this to match
> 
>    c. Click the 'Transform' button; this window allows you to build supercells from your .cif unit cell. Under 'Rotation Matrix', change the intergers to match the dimensions of the supercell you want to build. Click 'Okay', 'Yes', 'Add new equivalent positions to a list of symmetry operations', 'Okay'
> 
>    d. Click 'Apply' and 'Okay'; this larger menu often gets pushed behind other windows so make sure you find it!

At this point, you may notice that the .xyz file displayed in the VESTA terminal lists significantly more atoms than your supercell should contain (ex. 2x2x2 cell Barium Titanate should read 40 atoms, but at this stage will read 71 atoms in VESTA). This is because VESTA's default boundaries *include both faces of the supercell*, while convention only includes one. This means that any atom sitting on the boundary of your supercell will be drawn twice, and listed as two seperate positions in VESTA's terminal output. Thankfully, there is an easy way to fix this:

> 4. Go to the in-window Style menu, and click the 'Boundary...' button
>    
>    a. Under the 'Range of fractional coordinates', look for the x(max), y(max), z(max) fields
> 
>    b. Whatever integer value these fields currently display, decrease is *slightly* to a decimal (ex. if x(max) says 1, decrease it to 0.99); this will drop the maximum boundary just enough to exclude the duplicated atoms from the .xyz atom count
> 
>    c. Click 'Apply', then 'Okay'
>
> 5. To save, go to File > Export Data
>    
>    a. Name your file and double-check that XYZ format is selected
> 
>    b. Save, and you're done!

This will generate a cartesian .xyz that can be converted into fractional coordinates.

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
Run this script in VScode, Jupyter notebooks, or program of your choice.
