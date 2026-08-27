# Average Jobs
### Table of Contents
1. [Average Job Basics](#average-job-basics)
2. [Input File Structure](#input-file-structure)
3. [Submission File Structure](#submission-file-structure)
4. [Example Average Input File]()

## Average Job Basics
**Purpose:**

**Package:** average.x

**Resource/Time Usage:** low

An example [Average input file]() is attached for your reference (2x2x2 Tetragonal Barium Titanate).

## Input File Structure
```
1                      ! Number of files to read in
total_pot.dat          ! Name of file to be read in (from pp.x)
1.0                    ! Weight assigned to file
200                    ! Number of interpolation points along axis
3                      ! Direction of vacuum axis (1 = X, 2 = Y, 3 = Z)
0.0                    ! Size of macroscopic avergaing window (0.0 = standard planar average)
```

## Submission File Structure
```

```
