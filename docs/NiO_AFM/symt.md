# Setup
We will setup a larger cell for AFM NiO, this time we will use the labels
and also turn on spin polarization.

'''symt.inp
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

# Turn on spin polarixation
 spinpol
# Spin axis
spinaxis
  0.000000000000000   0.000000000000000   1.000000000000000  c

# Sites
atoms
4
  0.000000000000000   0.000000000000000   0.000000000000000   28 c up   # Ni up
 -0.500000000000000  -0.500000000000000   0.000000000000000   28 c dn   # Ni downn
  0.500000000000000   0.500000000000000  -0.500000000000000    8 c a   # O
 -0.500000000000000  -0.500000000000000   0.500000000000000    8 c a   # O
'''
New things! _spinpol_ turns on spin polarization, _spinaxis_ gives the axis along
which we will measure the spin (by default this is the z-axis, but make sure to specify it
ourselves). The spinaxis coordinates can be given in lattice (l) or cartesian (c) coordinates.
We use cartesian because it is easier in our specific case.

Note that we use labels 'up' and 'dn' labels are special, RSPt will generate spin polarized atomic
densities to start from. This way, we will get an antiferromagneic (AFM) setup for our
calculation. The _spinpol_ flag is very important to remember! The 'up' and 'dn' labels do
not tell RSPt to turn on spin polarization, so we need spinpol in order to properly read the
atomic densities. Otherwise, strange results can occur.

The Oxygen atoms all have the same label, because we don't care if RSPt finds symmetries
that make them all equivalent (which it should).
