# Relaxation Jobs


```
&CONTROL
  calculation = 'relax'             ! designates RELAX as the type of job to run
  prefix = 'NAME'                   ! labels and collects data associated with this job
  outdir = './tmp'                  ! where wavefunctions and other data is stored (directory must be created before running job)
  pseudo_dir ='/path/to/pseuds'
  tstress = .true.
  tprnfor = .true.
  etot_conv_thr = 1.0d-4  ! Ry
  forc_conv_thr = 1.0d-3  ! Ry/au
  nstep = 500                       ! max number of optimization steps allowed in job
  restart_mode = 'from_scratch'     !'from_scratch' = completly restarts, 'restart' = continue from where last job finished
/
&SYSTEM
  ibrav = 0                         ! cell type; 0 = free cell (determined by CELL_PARAMETERS)
  nat = NUM                         ! total number of atoms in your base cell/supercell
  ntyp = NUM                        ! total number of elements in your system
  ecutwfc = NUM                     ! kinetic energy cutoff for wfcs
  ecutrho = NUM                     ! kinetic energy cutoff for charge density, 4*ecutwfc is default
  occupations = 'smearing'          ! 'smearing' = smeared electrons for metals, 'fixed' = insulator with a bandgap (smearing converges faster, but fixed is needed for Ph.x)
  smearing = 'mv'                   ! type of smearing used
  degauss = NUM                     ! gaussian spreading in Brillouin zone integration
/
&ELECTRONS
  conv_thr = 1.0d-NUM               ! conv_thr can be low here as tight convergence isn't needed for structural verification
  mixing_beta = NUM                 !
  mixing_mode = 'plain'             ! 
  mixing_ndim = 16
  electron_maxstep = 1000           ! max number of iterations in SCF cycle
  diagonalization = 'david'         ! type of diagonalization used to process electronic Hamiltonian
/
&IONS
/
ATOMIC_SPECIES
    ATOM  ATOMIC_MASS  FILENAME     ! links to your pseudopotential files 
    ATOM  ATOMIC_MASS  FILENAME     ! all atom types in your cell must have a pseudopotential file listed here

CELL_PARAMETERS angstrom
    x    0.0    0.0                 ! dimensions of your unit cell/supercell (only needed for ibrav = 0)
    0.0    y    0.0
    0.0    0.0    z

ATOMIC_POSITIONS crystal
! Insert FRACTIONAL atomic positions here

K_POINTS automatic
   k1 k2 k3 0 0 0
```
