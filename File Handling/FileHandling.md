# File Handling

### Table of Contents:
1. [What is File Handling?](#what-is-file-handling)
2. [How QE Manages Files](#how-qe-manages-files)
3. [File Handling for Smeared vs Fixed Wavefunctions](#file-handling-for-smeared-vs-fixed-wavefunctions)

## What is File Handling?
In the context of this repository, file handling refers to how QE names, writes, and stores files containing information from your calculations. Understanding what information is written to which file is essential for everything from data extraction to job automation, and can resolve many job errors in input scripts.

File handling dictates what files are called during job submissions, and what files are created containing the results of that calculation. As quantum calculations often involve the handling of tens/hundreds/thousands of files, knowing where these files are stored and how the program calls them is very important when linking multiple jobs together.

## How QE Manages Files
QE handles files based on a few key parameters:

**prefix** - assigns a custom string to all generated folders and data files associated with the input file; **this is the most prevalent parameter for file handling in QE!** This parameter is found in the input file (.in) for a job. 

When running sequential jobs on the same system (ie relax --> SCF --> NSCF), *prefix* must be the same between all input files. All temporary, binary, and saved files are appended with a *prefix* string - this is how sequential jobs call the correct data from previous calculations. 

**outdir** - specifies the directory in which output data is stored. Found in the input file (.in) for a job.

For example, if *outdir = cat* and *prefix = mouse*, saved data would be stored in the directory *./cat/mouse.save*

*mouse.save* would contain the wavefunction files (wfc.dat), XML schema file (data-file-schema.xml), and charge density files (charge-density.dat). Temporary scratch files are also written to this directory.

**wfcdir** - specifies the directory from which initial wavefunction files are sourced. If not specified, defaults to *outdir*. Particularly useful in sequential jobs!

## File Handling for Smeared vs Fixed Wavefunctions
