# File Handling

### Table of Contents:
1. [What is File Handling?](#what-is-file-handling)
2. [How QE Manages Files](#how-qe-manages-files)
3. [File Handling for Smeared vs Fixed Wavefunctions](#file-handling-for-smeared-vs-fixed-wavefunctions)

## What is File Handling?
In the context of this repository, file handling refers to how QE names, writes, and stores files containing information from your calculations. Understanding what information is written to which file is essential for everything from data extraction to job automation, and can resolve many job errors in input scripts.

File handling dictates what files are called during job submissions, and what files are created containing the results of that calculation. As quantum calculations often involve the handling of tens/hundreds/thousands of files, knowing where these files are stored and how the program calls them is very important when linking multiple jobs together.

## How QE Manages Files

## File Handling for Smeared vs Fixed Wavefunctions
