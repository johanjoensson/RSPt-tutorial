# What? Why? How?
There is a problem in doing the LDA+U calculation the way we did in
session 3. We did a spin polarized DFT calculation, then added a
nonmagnetic double counting correction (all double counting corrections are
nonmagnetic), before finally adding a U to out d-orbitals. This means that
we have a DFT result with a certain spin spliting of the d-states, this spin splitting
is not removed by the double counting, then by applying the U we add even more spin
splitting! The end result is a spin splitting of the Ni d-states that is too large!

To solve this, at least partially, we can do non-spin polarized DFT. Then allow for
spin splitting only in the Greens function part of the code. To do this we set up
a nonmagnetic calculation, like we did in [session 3](../../session_3/NiO_AFM/nio_afm.md)
```
# Lattice constant in a.u.
lengthscale
7.8999286198

# Choice of MT radii
mtradii
3

# Lattice vectors (columns)
latticevectors
 0.00 -0.50 1.00
-0.50  0.00 1.00
 0.50  0.50 0.00


# Sites
atoms
4
  0.000000000000000   0.000000000000000   0.000000000000000   28 c a   # Ni up
 -0.500000000000000  -0.500000000000000   0.000000000000000   28 c b   # Ni down
  0.500000000000000   0.500000000000000  -0.500000000000000    8 c a   # O
 -0.500000000000000  -0.500000000000000   0.500000000000000    8 c a   # O
```
This time, we do not set up a spin polarized calculation! And, we make sure to keep
the Ni atoms separate by using different labels, `a` and `b`.

We also need to change one of the `prnt_array` values, set flag 11 to `T`. This
tells RSPt that even though the DFT part is not spin polarized, the density matrix
from DMFT will be.

Do the rest of the setup as usual.

## Rough SCF
Do a rough convergence of the SCF cycle, `fsq ~ 1e-6` or so.

## The very first LDA+U iteration
This time, we will do something special for the first LDA+U iteration, we will
force a splitting of the spin channels. Then we will converge the self-energy,
and end up with a spin split self-energy, without the DFT level extra spin split.
```
inputoutput
 F F  # read_sig csc

convergency
 1e-6 1e-4 999 1  # delta_n delta_sig max_it max_solver_it

tensmom
 Symbrk
 0  1  1  1.0

cluster
 1 UJ eV IdNiUp_d  # norb U? Identifier
 1 2 1 1 3 8.0 1.0  # t l e site basis U J
 2 2 0.65  0.10          # solver DC mixing symbrk

cluster
 1 UJ eV IdNiDn_d
 2 2 1 1 3 8.0   # t l e site basis U
 2 2 0.65 -0.10
```
The tensmom block does the initial spin splitting for us. Please read the manual
for an explanation of the black magic going on here. The `symbrk` value that has
appeared in each cluster is a scale factor to multiply the symmetry breaking thing
from the tensmom block. Setting it to `1.0` will usually result in MASSIVE spin splitting,
so a smaller value like `0.1` or smaller is usually a good idea. We set a positive value
for the cluster we want to have a net positive (spin up) magnetization, and negative values for the
negative (spin down) magnetization.

## All other LDA+U calculations
Once we have done 1 calculation with the `Symbrk` thing, we have our spin polarized self-energy.
We no longer need to force apart the different spin channels. So we change `green.inp`
```
inputoutput
 T T  # read_sig csc

convergency
 1e-6 1e-4 999 99  # delta_n delta_sig max_it max_solver_it

tensmom
 Symbrk
 0  1  1  1.0

cluster
 1 UJ eV IdNiUp_d  # norb U? Identifier
 1 2 1 1 3 8.0 1.0  # t l e site basis U J
 2 2 0.65  0.00          # solver DC mixing symbrk

cluster
 1 UJ eV IdNiDn_d
 2 2 1 1 3 8.0   # t l e site basis U
 2 2 0.65  0.00
```
Now we are ready to converge self-energies, and the full DFT cycle as well.

## Final remark
This method is how more advanced DMFT calculations should be performed. Keeping all
the spin polarization in the DMFT part, not the DFT part. Always do this unless you
don't want to.
