﻿# Main pipeline for any embed calculation using Rob wisdom and God's help

## 1. Prepare a CRYSTAL input (geometry, basis sets
https://www.crystal.unito.it/basis\_sets.html, etc) of the pristine unit cell.
Remember that this unit cell has to be big enough to contain all atoms that are
being added/removed/altered, so you may need to use a supercell. Also remember
CRYSCOR doesn’t support basis sets with pseudopotentials/ECPs yet
 
      a) You can use the vasp2crystal\_gen.py to make a slab from a vasp file,
      which gets rid of the vacuum from the 3-D-periodic plane-wave calc and
      gives you a 2-D-periodic geometry since a vacuum isn’t needed in CRYSTAL

      b) Atomic simulation environment (ASE)’s gui can read in/print out both
      vasp files and fort.34 files to write/read in CRYSTAL (see EXTERNAL and
      EXTPRT keywords). Once you’ve prepared a fort.34 file, you can run a
      quick CRYSTAL calc with STOP after the geometry to get the coordinates
      in CRYSTAL input format. The embedding workflow doesn’t use any fort.34
      files by default but could easily be modified to

      c) XCrysDen can read CRYSTAL input files and help you manipulate
      geometries. You can use this to easily exploit functionality in crystal
      like SLABCUT, ATOMINSE, and SUPERCEL to put your geometry together. It
      can be finicky

## 2. Prepare the geom\_mod file, remembering that geometries along periodic axes
are scaled by the lattice vector. Use keywords like “substitute, add, remove,
move, and vacancy” to manipulate the pristine CRYSTAL input

## 3. Submit the embedding workflow using the make\_inputs2.py script, checking if
it has arguments to intake or just requires you to name the CRYSTAL input as
“INPUT”. If it’s a non-trivial calc, you’ll want to submit it in the background
with “nohup python3 make\_inputs2.py &”

## 4. Wait a while, especially if you haven’t parallelized the CRYSTAL calcs (this
isn’t done by default to avoid queue times)

## 5. Enter the Frag directory and ensure the job ran successfully. It might error
out if a basis set for an element is missing, HF2 job failed due to basis set
linear dependence, etc. Troubleshoot if necessary

## 6. If all is well with the Frag job, search the output for “MANFRAG”. The table
above the MANFRAG table will help you pick a fragment based on the manipulated
atoms and their neighbors. In general, it’s correct to expand the fragment
until your energy differences of interest in Molpro (or Cryscor if HF or MP2)
stop changing in Step 8. This means your fragment size is big enough. If you
want to visualize the fragment to make sure you’ve picked the right one, search
for “xyz” in the output and make sure you copy the atom coordinates of the
fragment after manipulation

## 7. Copy the lines FCIINT, MANFRAG, number of atoms, and the lines for the atoms
themselves into a copy of frag.inp, in a separate directory. Also copy Frag.job
to this directory. Add to the new frag.inp also the keywords “FRAGHF” and
“FRAGDUMP”. There are also other keywords you may need for SCF convergence,
altering # of electrons, etc. Here I give give only the bare minimum keywords.
Also, if you have an open-shell fragment with an odd number of electrons, alter
the number of electrons with “DELTAEL” to make it even-numbered. There are
still some bugs in UHF, so we’ll use Molpro ROHF to re-rotate the RHF solution
later instead. Submit the job

## 8. Once the previous step’s job has converged, you’ll get a FCIDUMP file (or if
you only wanted RHF or RMP2, you’re done, see output). The core is frozen in
the printing of this FCIDUMP file by default. Put this in a directory with the
job.run and posthf\_inp\_den.py and then run “python3 posthf\_inp\_den.py FCIDUMP”.
In posthf.inp, modify the “set\_nelec=” and “set,spin=” if necessary, especially
if you followed the DELTAEL thing from Step 7 to avoid UHF. If you want
CCSD(T), this is in posthf.inp by default and you can just submit the molpro
job. Otherwise, set up CASSCF (multi), etc after the HF lines, using molpro
normally. Remember that these FCIDUMP-based molpro calcs will only give you
energies; don’t try obtaining response properties etc or the numbers will be
all wrong. Submit the job and enjoy your embedded fragment results!


