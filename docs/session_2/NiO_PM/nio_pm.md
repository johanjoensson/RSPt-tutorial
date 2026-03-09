# symt.inp
```
# Lattice constant in a.u.
lengthscale
7.911031407234099
# Lattice vectors (columns)
latticevectors
  0.500000000000000   0.500000000000000   0.000000000000000
  0.500000000000000   0.000000000000000   0.500000000000000
  0.000000000000000   0.500000000000000   0.500000000000000
# Sites
atoms
2
  0.000000000000000   0.000000000000000   0.000000000000000   28 l a   # Ni
  0.500000000000000   0.500000000000000   0.500000000000000    8 l a   # O
mtradii
 3
```
This file should look very familiar, the only big difference is that we now have 2
atoms in the basis, Ni and O. Note that we use the same label "a" for both Ni and O,
don't worry, RSPt won't think that they are the same atom, because the atomic numbers are different.

Run `sumt -all` to generate the default configuration files.


# Running atom
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Atomic-densities))

Just type `make`, to create `atomdens` in the main directory .


# Running cub
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Meshing-with-the-k-points))

This time we will use a coarse k-point mesh, because we want the calculations to be fast.
Use a `5x5x5` k-point mesh (for the superlattice), with a shift of `1/2 1/2 1/2`. Also generate
the tetrahedra.

Copy/link the `cub.k.5` to `spts`, `cub.t.5` to `tetra`.

# Change some settings
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#The-basics-of-the-basis))

Once again we will use the linear tetrahedron method for Brillouin zone integration.
We want to use only negative tails. And maybe change some linearization flags? This time there
are 2 element files, one for Nickel and one for Oxygen. Check to see what states are included in the
LMTO basis, and which are put in the core for the different atoms.

Copy the `length_scale` and `strain_matrix` files to the main calculation directory and create the data
file.

# Converge SCF
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Running-RSPt))

Converge the SCF cycle, check for errors and update the settings if needed
(Fourier mesh, linearization flags, mixing, etc.). NOTE: this time we have
two different types of atoms, so we get more data to check.

# Plot DOS and pDOS
This time, we will do some light postprocessing. Since we are using the
linear tetrahedron method we can ask RSPt to calculate the DOS and pDOS
for our system. We do this by changing 2 flags in the `prnt_array`. Flag
`65` tells RSPt to calculate the DOS, and flag `64` (in combination with `65`)
calculates the pDOS. Change the flags and run one iteration of RSPt
(advanced users can tell RSPt to stop after calculating the eigenvalues,
if they want).

## Reading the files
If everything went well, RSPt will have produced a couple of new files.
The file `Dos.hea` contains a summary of the settings for the DOS calculations.
The file `Dos.0` contains the total density of state, and the integrated density of states.
The first column is the energy, second column total DOS, and third column the integrated DOS.

Plot the DOS, is the system metallic or insulating? Where is the Fermi energy? (tip, search for
`fermi energy` in the out file)

That's the total DOS, but what about the partial (or projected) DOS? Of course the pDOS
can be found in the PDos files. `PDos.01.1` contains the pDOS for type 1 (check the data file
to see what atom(s) type 1 corresponds to), `PDos.02.1` contains the pDOS for type 2.
Here, the columns are `E`, `s`-orbital, `p`-orbital, `d`-orbital, `f`-orbital, higher `l`-quantum numbers.
The file `PDos.int.1` contains the pDOS corresponding to the interstitial. What states are close to the
Fermi energy in our calculation?

## More on DOS and pDOS
In the manual there is more information on how to calculate the DOS and pDOS in the LMTO basis.
We will soon use a different way to calculate the DOS and pDOS.

There is a method for calculating the electronic band structure using the LMTO basis only, but it is
somewhat involved, and a bit limited. We will use a different way to calculate bands later.
