# Launching kDNA border simulation

The folder `/data/init-files` contain the starting files necessary to run kDNA
border simulations.  kDNA is modelled as a circular patch of hexagonal lattice
made of 604 rings. The border is identified as the set of rings that have only
2 linked neighbours, plus all rings directly linked to these. In each
simulation the border is constrained to have a certain radius of gyration with
the lammps fix spring command.  This allow to account for the fact that in
reality the kDNA border is shorter than the perimeter of the adsorbed kDNA,
since several of its rings become elongated upon adsorption.


# Folders contents
The folders are created programmatically. Each folder contains the following
files, renamed according to the target radius of gyration `rg`. The parameters
in the files are changed accordingly.
```
.
├── n0604_rg0.10_rgK03.00.dat      # --> Initial datafile storing network topology
├── n0604_rg0.10_rgK03.00.lmp      # --> basic Lammps parameter file defining the sim. variables and steps
├── minimize.lmp                   # --> lammps minimization step, called during the first run (head.pbs)
├── output_append.lmp              # --> lammps outpus
├── saw_fene_polymer.lmp           # --> lammps definition for Kremer-Grest polymers
├── chain.sh                       # --> bash script to launch a chain of jobs on a PBS scheduler
├── head.pbs                       # --> first job (includes minimization)
└── restart.pbs                    # --> restarts (avoid minimization)
```

# Modifying the simulations
To modify how the simulation behave, open `n0604_rg0.10_rgK03.00.lmp` and
change the following parameters. Note that all of these can be overwritten by
`restart.pbs` and `head.pbs`. The one provided in the .lmp file are just the
defaul values.
```
variable seed index 168316          # random seed (overwritten when calling lammps)
variable taudamp index 10           # Langevin damping time
variable dt equal 0.01  	    # timestep (must be lower than 0.0124
variable rgspring index 3           # K spring
variable rgtarget index 23.733      # target rg for the border
variable time index "23:30:00"      # walltime to stop the simulation (overwritten by head.pbs and restart.pbs)
variable restart index 0            # 0: start from datafile . 1: start from latest restart file. (overwritten when calling)
variable minimize index 0           # 0: do not minimize. 1: minimize (overwritten when calling)
variable name index "n0604_rg0.10_rgK03.00"
variable dumptime index 10000000    # write a snapshot ever dumptime steps
variable restime  index 50000000    # write a restart every restime steps
variable runtime  index 1000000000  # total runtime 
```

# Running the simulation
The main lammps file expects the `datafile` variable in input, so it should be
called with e.g.
```
mpirun -np 24 $LMP  -v time ${TIME} -v minimize 1 -v datafile n0604_rg0.10_rgK03.00.dat -in n0604_rg0.10_rgK03.00.lmp >> log
```

The files `.sh` and `.pbs` in the folders create a chain of batch jobs on an SGE batch scheduler and automatically modify
both the random numbers and the other parameters needed by the lammps simulations.
