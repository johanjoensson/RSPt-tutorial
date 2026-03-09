# Generate the k-point mesh
Again, a rough k-point mesh. _5x5x5_ with a shift of _1/2 1/2 1/2_, and tetrahedra.

Copy/link as before.

# Band structure path
This time, we will also calculate the band structure, so we need to set up a path in
k-space. We will use the program kpath to generate the path. kpath requires an input
file, called _kpath.inp_.
'''kpath.inp
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
'''
Then, just run _kpath_ to generate the _spts.band_ file.
