# File Handling

### Table of Contents:
1. [What is File Handling?](#what-is-file-handling)
2. [How QE Manages Files](#how-qe-manages-files)
3. [File Handling for Different Jobs](#file-handling-for-different-jobs)

## What is File Handling?
In the context of this repository, file handling refers to how QE names, writes, and stores files containing information from your calculations. Understanding what information is written to which file is essential for everything from data extraction to job automation, and can resolve many job errors in input scripts.

File handling dictates what files are called during job submissions, and what files are created containing the results of that calculation. As quantum calculations often involve the handling of tens/hundreds/thousands of files, knowing where these files are stored and how the program calls them is very important when linking multiple jobs together.

## How QE Manages Files
QE handles files based on a few key parameters:

**prefix** - assigns a custom string to all generated folders and data files associated with the input file. This parameter is found in the input file (.in) for a job. 

When running sequential jobs on the same system (ie relax --> SCF --> NSCF), *prefix* must be the same between all input files. All temporary, binary, and saved files are appended with a *prefix* string - this is how sequential jobs call the correct data from previous calculations. 

**outdir** - specifies the directory in which output data is stored. Found in the input file (.in) for a job.

For example, if *outdir = 'cat'* and *prefix = 'mouse'*, saved data would be stored in the directory *./cat/mouse.save*

*mouse.save* would contain the wavefunction files (wfc.dat), XML schema files (data-file-schema.xml), and charge density files (charge-density.dat). Temporary scratch files are also written to this directory.

**wfcdir** - specifies the directory from which initial wavefunction files are sourced. If not specified, defaults to *outdir*. Particularly useful in sequential jobs!

## File Handling for Different Jobs
### Smeared and Fixed SCF
When working with smeared and fixed occupations (see [Smeared vs. Fixed Occupations](https://github.com/kylayounger/So-You-Want-To-Model-Materials-In-Quantum-Espresso/blob/main/Self-Consistent%20Field%20(SCF)%20Jobs/Smeared%20vs.%20Fixed%20Occupations.md)), it is important to keep smeared/fixed wavefunction files saved in separate directories. 

In the input file for the smeared SCF calculation, set *outdir = 'smear_data'.* This will save the smeared wavefunctions in *./smear_data/prefix.save* when the calculations completes.

In the input file for the fixed SCF calculation, set *wfc_dir = 'smear_data'* and *outdir = 'fixed_data'.* This will allow QE to start the fixed SCF calculation using the previously-converged wavefunctions found in the *./smear_data/prefix.save* directory, and save them in a separate directory, *./fixed_data/prefix.save*, without overwriting the original smeared data.

### NSCF and Bands
NSCF and Bands jobs implement a denser k-point mesh than the original SCF calculation used. Because of this, it is important to separate the NSCF and Bands wavefunction files from the SCF wavefunction files. This preserves the k-mesh of the original SCF wavefunctions so they can be used in future calculations (such as Phonon) without having to be reprocessed.

In the input file for the NSCF calculation, set *wfcdir = 'fixed_data'* and *outdir = 'nscf_data'.* This will allow QE to start the NSCF calculation using the SCF wavefunctions found in the *./fixed_data/prefix.save* directory, and save them in a separate directory, *./nscf_data/prefix.save*, without overwriting the original SCF data.

Additionally, some implementations of NSCF require a smearing parameter - saving the NSCF wavefunctions in their own *outdir* ensures that the fixed SCF wavefunctions remain unedited for subsequent calculations.

