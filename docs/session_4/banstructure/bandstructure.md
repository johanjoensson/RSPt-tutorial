# Bandstructure
As you might have guessed by now, plotting band structures in RSPt
can be done, but it is perhaps not the easiest thing to do. In this
section I will describe how to use the Greens function formalism to
calculate the bands (or the spectral function).

## Generate the kpoint path
First we need a path in reciprocal space along which to plot our bands.
The program `kpath` offers a nice way to set this up. `kpath` expects an input file
called `kpath.inp`.
```
# File for generating a set of lines through k-space for
# band structure plotting. First you we must write
# the bravais lattice vectors, i.e. the same vectors
# as in your symt.inp file.
# The following lines define the path of high-symmetry
# points to follow in the BZ according to
# k1 k2 k3 "Symmetry symbol" "[Number of k-points within the line]"
# Specifying the number of k-points is optional. If it is
# not specified it will be given the same k-point density
# as the first line (which is recommended).
#
# Note that in kpath each high symmetry point specified
# below is included only once in the spts file.
#
# Bravais lattice basis for fcc:
0.00000000000000 0.50000000000000 0.50000000000000
0.50000000000000 0.00000000000000 0.50000000000000
0.50000000000000 0.50000000000000 0.00000000000000
#
# High symmetry k-points (in the reciprocal basis)
0.50000000000000 0.00000000000000 0.50000000000000 X 60
0.00000000000000 0.00000000000000 0.00000000000000 G
0.50000000000000 0.50000000000000 0.50000000000000 L
0.62500000000000 0.25000000000000 0.62500000000000 U
0.50000000000000 0.00000000000000 0.50000000000000 X
0.50000000000000 0.25000000000000 0.75000000000000 W
0.50000000000000 0.50000000000000 0.50000000000000 L
0.37500000000000 0.37500000000000 0.75000000000000 K
0.50000000000000 0.25000000000000 0.75000000000000 W
```
With this input file, running the command `kpath` will create the files
`spts.band` and `kpath.log`.

## Set up green.inp
We can now set up the `green.inp` file for calculating the band structure.
It is quite similar to what we did for DOS/pDOS
```
energymesh
 1001 -1.0 1.0 0.010  # nemesh  emin emax eim

mixing
 1  0.15  0  EF

spectrum
 Band Pband eV
    60    70    29    25    29    22    11    18
 X     G     L     U     X     W     L     K     W

cluster
 1 0 IdNi_d  # norb U? Identifier
 1 2 1 1 3   # t l e site basis
```
The spectrum block now looks weird, right? This is because we need to specify
a bunch of information about the kpoint path we are using. Thankfully, at the end of the
`kpath.log` file, the suggested `spectrum` block to use is printed, so we just copy that.

Also, there is now a block called mixing! This contains settings for how to update the self-energy,
and also (optionally) the Fermi energy to use. For a band structure plot, only the Fermi energy value is important,
since we are not updating any self-energies. Replace `EF` with the value of the Fermi energy of your calculation
(`fermi energy` in the output file).

## Plotting bands
Run one iteration of RSPt, hopefully all went well and you get a bunch of files, such as `band.gpi`, `pband-Ni.gpi`.
Files ending with `.gpi` are `gnuplot` plotting scripts. So, run `gnuplot bands.gpi`, to produce the file `band.pdf`, which contains
the band structure plot. The `pband-*.gpi` files plot the projected bandstructure and puts the plots in the corresponding `.pdf` files.
The spectrum flag `Eps` prevents the creation of the `.pdf` files, and instead creates a bunch of `.eps` files
with all the bandstructure plots and projections. More information and details are available in the manual.
