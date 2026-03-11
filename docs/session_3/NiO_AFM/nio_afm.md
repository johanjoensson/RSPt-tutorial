# symt.inp
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

spinpol
# Spin axis
spinaxis
  0.000000000000000   0.000000000000000   1.000000000000000  c

# Sites
atoms
4
  0.000000000000000   0.000000000000000   0.000000000000000   28 c up   # Ni up
 -0.500000000000000  -0.500000000000000   0.000000000000000   28 c dn   # Ni down
  0.500000000000000   0.500000000000000  -0.500000000000000    8 c a   # O
 -0.500000000000000  -0.500000000000000   0.500000000000000    8 c a   # O
```
We have some new things in the `symt.inp`, `spinpol` tells RSPt to do a spin polarized calculation,
`spinaxis` sets the direction of the spin quantization axis. Just as with atomic coordinates we can
give the direction in cartesian coordinates (`c`) or lattice (`l`). In our case we will set the `z`-axis as
our spin quantization axis (it really does not matter what we pick, unless we start including spin-orbit coupling effects).
This time, we have even more atoms in the unit cell! 2 Ni and 2 O. The 2 Ni atoms
have different labels, `up` and `dn`, this tells `symt` that we want to generate spin polarized atomic densities, net spin up for the `up` label, and net spin down for `dn`

Run `symt -all` to generate the default configuration files.


# Running atom
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Atomic-densities))

Just type `make`, to create `atomdens` in the main directory. No need to change anything, since we used the `up` and `dn` labels.


# Running cub
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Meshing-with-the-k-points))

This time we will use a coarse k-point mesh, because we want the calculations to be fast.
Use a `3x3x3` k-point mesh (for the superlattice), with a shift of `1/2 1/2 1/2`. Also generate
the tetrahedra.

Copy/link the `cub.k.3` to `spts`, `cub.t.3` to `tetra`.

# Change some settings
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#The-basics-of-the-basis))

This time, we will use the Fermi smearing method for Brillouin zone integration.
```
 (i6)
     0
 (/ 2f12.0, i6)
       W(Ry)        dE/W     .
      0.0025         64.     0
```
The first integer is 0, for Fermi smearing. `W` gives the smearing, in units of Ry. `dE/W` should be kept constant, ish.
Since we decreased `W` by a factor 10, we need to increase `dE/W` by a matching factor, ish. We go from 4 to 64, roughly a factor of 10,
it's really not that crucial.

The element files will look a bit different now.
```
 (/ i6, 2f12.0, 5x, a1 /)
     n           S          dx coord
   543  .826983014       0.025     v
   543  2.02640800       0.025     a
 (/f6.0, i6, f12.0, 2f6.0, 2x, 2i1, f2.0, 11x, a1//(2i6, f12.0, i12))
     Z    nc      Sinf/S     . Sws/S  ....     ScoFlag
   28.     7          1.    1.    1.  10.0           p
     n     k         occ        flag
     1    -1          2.           0
     2    -1          2.           0
     2     1          2.           0
     2    -2          4.           0
     3    -1          .0           0
     3     1          .0           0
     3    -2          .0           0
    12 Bases
     0     1     1
     0     1     2
     0     1     3
     1     1     1
     1     1     2
     1     1     3
     2     1     1
     2     1     2
     0     2     1
     0     2     2
     1     2     1
     1     2     2
     4     4     3     4     5     6     7     8     9
    20    21    -1     0     0     0     0     0     0
     4     4     3     4     5     6     7     8     9
    20    21    -1     0     0     0     0     0     0
     3     3     3     4     5     6     7     8     9
    -1    -1     0     0     0     0     0     0     0
     3     3     3     4     5     6     7     8     9
    -1    -1     0     0     0     0     0     0     0
```
Double the linearization flags! Two lines (spin down and spin up) for energy set 1, 2 for energy set 2.
Same rules apply as before. The flags 20, 21, etc. will orthogonalize to matching spin states in the other energy set.

Make changes in files as you see fit (`tail_energies` perhaps?).

Copy the `length_scale` and `strain_matrix` files to the main calculation directory and create the data
file.

# Converge SCF
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Running-RSPt))

Converge the SCF cycle, check for errors and update the settings if needed
(Fourier mesh, linearization flags, mixing, etc.). This time each iteration will take a bit of time, because we have more atoms than before.

# Plot DOS and pDOS, but with Greens functions
We are using Fermi smearing this time, which means we cannot use the LMTO method for calculating the DOS/pDOS (beacause of bad reasons...).
Instead we will use RSPt's Greens function framework (which is what we use for bandstructure, LDA+U, LDA+DMFT, etc.). To turn on the Greens function
machinery we need to create the input file `green.inp`.
```
energymesh
 1001 -1.0 1.0 0.010  # nemesh  emin emax eim

spectrum
 Dos Pdos

cluster
 1 0 IdNiUp_d  # norb U? Identifier
 1 2 1 1 3   # t l e site basis

cluster
 1 0 IdNiDn_d
 2 2 1 1 3
```
This file uses a bunch of input blocks to set everything up.
`energy mesh` gives the energy range we will calculate the DOS/pDOS over (the Fermi energy is a 0).
Our mesh will consist of 1001 points evenly spaced over the energy range -1 Ry to 1 Ry. The final number is a smearing parameter (not the Fermi smearing parameter), smaller values give smaller smearing, but too small smearing will give you crap output. Usually 0.010 is a good starting point.

The spectrum block contains the data we want to calculate for plotting. `Dos` means DOS, `Pdos` means pDOS (orbitally resolved, spin resolved, etc.).

The cluster blocks contains the sets of orbitals that we want to do the while Greens function thingy with.
The first line in each cluster contains the number of orbitals in that cluster (veeeeeery rarely is this anything other than 1),
the second number is related to how our Coulomb U, we put 0 because we don't want any U.
The third line specifies the LMTO basis function to do Greeny things to. `t` is the type, in our case types 1 and 2 are Ni, type 3 is O. `l` is the
`l`-quantum number of the LMTO basis function we want to select, `e` is the energy set that contains this basis function. `site` specifies which
site (if out type contains several equivalent sites) we want to look at. The site is always 1, unless you have gooooooood reasons for why it is not.
`basis` selects what definition of localized orbitals to use, the manual explains the different choices very well. `0` always means spherical harmonics,
for `d`-electrons `3` means cubic harmonics (eg, t2g). Check the manual for more info. We use 3, because we have a cubic system which should give us nice eg and t2g
states.

We haven't talked about the Id thingy on the second line of each cluster, that is because it is simply a nice feature. By default each cluster gets and id composed of the `t l e site basis` values, but this
is not always very readable, so we can give each cluster it's own _unique_ name.

Simply run 1 iteration of RSPt, check the out file for the new steps the code takes (search for `BRIANNA`, because of reasons...).

The code should now have produced files called `dos.dat` (DOS for the entire cell) and `pdos-NiUp_D.dat`, `pdos-NiDn_d.dat`, guess what they contain. The
files contain a header explaining what each column represents. Plot them and check what states we have in our valence.

Where is the Fermi energy in the DOS?

## Add O cluster
Add a cluster for the O atoms valence electrons and calculate the (p)DOS where are the O p-states?
