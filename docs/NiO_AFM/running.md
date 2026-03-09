# Converge SCF
Make sure to check for errors! This time, the convergence will be a bit slower
(larger cells tend to be harder to converge), and each iteration might be slower
as well (more atoms, and 2 spin directions). So, if you can, use mpi to run RSPt
(e.g. mpirun -n 2 rspt, to use 2 CPU cores).

# Plot the DOS and pDOS
Just as before, tell RSPt to calculate DOS and pDOS. Note that we now get
two PDos files for each element, PDos.01.1 for type 01, spin projection 1,
PDos.01.2 for type 01, spin projection 2. Is the system still metallic?
What states are near the Fermi energy?

# Change the brillouin zone integration
We will now introduce a different way to calculate the density of states.
This other method requires that we use Fermi smearing for Brillouin zone
integration. Edit the file _dta/brillouin_zone_integration_.
'''brillouin_zone_integration
 (i6)
     0
 (/ 2f12.0, i6)
       W(Ry)        dE/W     .
      0.0025         64.     0
'''
The first integer is no 0, which means Fermi smearing. The W parameter is
the smearing parameter (k_B * T), our value will be close to room temperature.
The dE/W parameter needs to be changed to match the change in W. W was divided
by 10, so dE/W should be changed by roughly the inverse (here we go from 4 to 64,
which is a factor of 16, slighlty larger than 10. Don't worry, it's okay).

Also change the prnt_array to remove the LMTO DOS calculations (this method requires the
linear tetrahedron method). Set flags 65 and 64 to false.

# Prepare the new input file
The new method we will use relies on calculating the Greens function and
projecting it onto some localized ('atomic like') orbitals. The input file
for this is called green.inp
'''green.inp
energymesh
 1001 -1.0 1.0 0.010 # num_mesh_points e_min e_max smearing

spectrum
 Dos Pdos

cluster
 1 0 IdNiUp # Num_sites 0 Id_tag
 1 2 1 1 3  # t l e site basis

cluster
 1 0 IdNiDn
 2 2 1 1 3
'''
The energy mesh parameter determines the energy range we want to calculate the DOS over.
The spectrum parameter contains a list of plottable things to calculate, we want DOS and
pDOS.
The two cluster blocks define our atomic like states to project onto. We want to start
from the LMTO basis functions and extract orbitals with d-character for the 2 Ni types.
The first row in each cluster starts with a 1, because we have only one set of orbitals per
cluster (this is true always, always, always. Except for very specific and rare cases). The 0
just means that we don't do any DMFT stuff (for now). t, l, e, site determine the LMTO orbital
character we are interested in, type 1 is one of the Ni types, l is 2 for d-electrons, e is the
energy set containing the orbitals (1 for us), and basis is the basis in spherical harmonics
basis we want to use (3 gives us cubic harmonics, which for cubic systems gives us eg and t2g).
This is a bit complicated, but trust me, it is waay easier in the long run.

# Calculate the DOS
Run one iteration of rspt.
