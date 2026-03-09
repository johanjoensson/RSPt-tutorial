# Our first RSPt calculation
We will start by doing a very simple calculation, with minimal changes to the input.
We will not be doing any postprocessing, or fancy plotting. That comes later in the tutorial.

Create a new folder for your RSPt calculation.

## Set up the geometry and initial configuration
RSPt uses a program called `symt` in order to set up the geometry and initial configuration for the calculation.
symt assumes that it is run inside a folder called `sym`, so create that folder now.

Inside the sym folder, symt will read an input file called `symt.inp`, so lets create it now.

```bash
# Lattice constant in a.u.
lengthscale
 6.5670738518462075
# Lattice vectors (columns)
latticevectors
  0.500000000000000   0.500000000000000   0.000000000000000
  0.500000000000000   0.000000000000000   0.500000000000000
  0.000000000000000   0.500000000000000   0.500000000000000
# Sites
atoms
 1
  0.000000000000000   0.000000000000000   0.000000000000000   28 l a   # Ni
# Method to set MT radii
mtradii
 3
```

## Understanding the input file

RSPt uses (Rydberg) atomic units, that means distances are measured in atomic units (a.u.)
and energies are measured in Rydberg (Ry).

symt.inp can be written in two different formats, in this tutorial I will use the `new` format.
Here we use labels in separate lines to divide up the input. The order of the labels is arbitraty.
But, in the new format, the labels are mandatory.
Comments use the `#` character, and after a `#` the rest of the line is simply ignored.

The `lengthscale` can be any positive number really, your lattice vectors will be scaled by the
lengthscale. In this example we set the lengthscale to the lattice parameter of the FCC Ni lattice.
lengthscale is mandatory.

The `latticevectors` block specifies our lattice vectors as a 3x3 matrix. Our lattice vectors are the
`columns` of this matrix, the first column is our first lattice vector (or a), the second column is
the second lattice vector (or b), and the third column is the third lattice vector (or c).
latticevectors is mandatory.

The `atoms` block specifies the basis (position of the atoms) of our crystal.
The first line is the number of atoms in the basis (in our case 1, because we have 1 atom).
Then comes one line per atom, specifying it's coordinates, atomic number (Z), how to read the coordinates, and finally a label.

The coordinates are 3 floating point numbers. RSPt uses a very strict precision for determining symmetries etc. so it is good
practise to give positions with 15 decimal places (otherwise you might get fewer symmetries than you expect, and therefore a
slower calculation).

The atomic number is how RSPt identifies what type of atom this is, Ni has atomic number 28.

The coordinates can be given in lattice (reduced) coordinates by specifying `l`, or cartesian coordinates by specifying `c`.
Reduced coordinates give the position in terms of the lattice vectors (a, b, and c), and cartesian coordinates give the position in terms of the cartesian axes (x, y, and z).
There are more options available, but we will not cover them here.

Finally, the label is just a way for us to indicate to RSPt wheter we want to be able to distinguish atoms in our calculation or not. It can be any alphabetical character
(or short string), usually we use a, b, c and so on. The label is only relevant if we have more than one atom of a kind (2 Ni atoms for instance), but it must always be specified.
More on this in a later session.

`atoms` is mandatory.

The `mtradii` block tells RSPt what algorithm to use for setting the Muffin-Tin radii of the atoms.
This is an important part for the LMTO method. The muffin tins must NOT overlap, the calculations will crash. Since RSPt is
a full potential code it can handle muffin tins with rather large distances between neighboring muffin tins, but this is usually a bad idea for other reasons.
The default setting of 1 uses an algorithm that might generate overlapping MT, and then we have to manually decrease them so that they no longer overlap. A better way is
to simply use option 3, which tries to generate MT that cover 95% of the distance between nearest neighbors, thus completely avoinding
overlapping MTs, and still keeping the interstitial region between MTs small.
`mtradii` is optional, but highly recommended.

## Running  symt
Now that we have out input file set up, we are ready to run `symt -all`. Do that.

New files have been generated in the sym folder, of these `symt.out` is the most
interesting one. It contains information about what symmetries symt was able to find
and what euler angles it used. This is rather advanced stuff, so let's move on


# Atomic densities
We need a decent starting guess for our DFT calculation.
RSPt uses overlapping atomic densities for this. The input etc is stored in the folder `atom`.

## Running atom
Just run `make` and it will generate the starting atomic densities for us. The starting densities are
stored in the file `atomdens`, in the main calculation directory.

You can change things in the data files in this folder, if you want to. But we won't
because it is outside our current scope.

# Meshing with the k-points
By default RSPt tries to use a generalized Monkhorst-Pack grid.
This means generating a cubic superlattice, generating the kpoint mesh
for the superlattice, and finally transforming the kpoints back onto the
original lattice. The implementation in RSPt is not perfect, so it may fail in finding a good superlattice.
In this case cub will not generate a kpoint mesh, and will ask you to change a matrix in an input file to the unit matrix.

## Files read by cub
When we ran `symt -all` it generated a bunch of files for us. One of these is `cub.inp`, the input file read by cub.
This file contains two matrices. R is the matrix we specified in symt.inp, it defines the lattice (up to some scaling factor).
M is the matrix used to generate the superlattice for the generalized Mokhorst-Pack scheme. Setting this matrix to the unit matrix
results in the regular Mokhorst-Pack scheme for kpoint generation.

## Running cub
The program `cub` is an interactive program that will ask you for input in order to generate the kpoint mesh.
First (usually) it will ask you for the dimensions of your kpoint mesh, in our case let's use a 5x5x5 mesh (this is very coarse,
but this is just a tutorial run). Just enter `5 5 5` and press enter. Next, cub will ask you about a shift vector, this can be used to generate
a mesh that does not contain the high symmetry points of the Brillouin zone. For cubic systems a shift of (1/2,  1/2, 1/2) is good. We specify the shift
we want by providing first the numerators, `1 1 1` in our case. Then we specify the denominators `2 2 2` for us. If we don't want a shift
we can specify `0 0 0` for the numerators, and any nonzero integer for the denominators, e.g. `1 1 1`.

If all goes well, cub will generate the k-point mesh for us. If not (e.g. you gave an incorrect shift) it will tell you what went wrong. With the k-point mesh done, cub
will ask if we want to generate tetrahedra, used by the linear tetrahedron method, in our case the answer is yes, we want the tetrahedra.
Then finally, cub will ask if we are done, maybe we want to generate several different k-point meshes. We are done, so answer yes.

## Preparing the files for RSPt
In the bz folder, cub should have generated the files `cub.k_5` and `cub.t_5`, these are the kpoint mesh and tetrahedra. We now need to tell RSPt which k-point mesh and tetrahedra
we want to use (remember cub asked if we wanted to generate more k-point meshes). To do this, change into the base calculation folder and run the command ` link_spts 5 `. Or, just copy the
`bz/cub.k_5` file to spts, and `bz/cub.t_5` to tetra.

That's it! We have k-points and tetrahedra!


# The basics of the basis
The dta folder contains a load of files. We won't touch many of them.
All the files are in Fortran fixed format, which means a misplaced space
or anything really, can cause catastrophic failure. Edit these files carefully!
(in vim, replace mode is recommended)

## Brillouin integration
```
 (i6)
     1
 (/ 2f12.0, i6)
       W(Ry)        dE/W     .
       0.025          4.     0
```
This file contains information about how we integrate quantities over the brillouin zone.
First there is a line telling Fortran that the next line contains a single integer.
That single integer is 1, this means linear tetrahedron method. If we use the linear tetrahedron method,
the rest of this file is ignored. If we use Fermi smearing however, it is not (and, later on we will).

## Element
The element_28 file contains information about the LMTO basis functions for the Ni atoms. If we have other
atoms in the cell, they will have their own element file. For now, let's just look at it
```
 (/ i6, 2f12.0, 5x, a1 /)
     n           S          dx coord
   543  .859609310       0.025     v
   543  2.20609127       0.025     a
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
    20    21     0     0     0     0     0     0     0
     3     3     3     4     5     6     7     8     9
     0     0     0     0     0     0     0     0     0
```
First we have a Fortran format starting with a slash, this tells fortran to skip
a line, then it will read an integerm two floats, 5 spaces and a character, then finally skip another line.
(This is fun, right?)
We only touch the first floating point. This sets the MT radius for the Ni atoms. The first line gives the
muffin tin radius as a fraction of the unit cell volume (add ^(1/3) or ^(3) wherever you want). This is the default
way. The second line gives the MT radius in atomic units directly, remember that by default this line is skipped.
If we want to change the MT radius, we do that here. Remember to update the Fortran format to skip the incorrect
line, and read the correct one.

Next comes a section where we define what electronic states are treated as core states, and thus are not
part of out LMTO basis. Lets leave this as it is (you can chage stuff here, if you are advanced).

The line reading "12 basis" is the start of the specification of the LMTO basis. Our Ni atoms will have 12
basis functions each. The 12 lines with three integers in each tells RSPt the l quantum number of these basis
states, which energy set they belong to, and finally what kinetic energy tail they should have. This is all advanced
and complicated, so let's leave it.

Then finally we have 4 line with loads of integers. There are 2 lines per energy set, and since we have 2 energy sets
we get 4 lines. The first 2 lines correspond to the first energy set (whic contains the first 8 basis functions).
The upper row gives the n-quantum number for these basis functions, in our case the first energy set contains basis
functions for 4s, 4p, and 3d. The 4f, 5g, etc. are used to set up the basis. Let's skip that.

The lower row tells RSPt how we want to select linearization energies. 0 means we use the "band center" as linearization energy,
this is usually good but sometimes it causes problems. -1 means we want to use the "center of the occupied band", which is very stable
and reliable. If 0 fails, then -1 will work. The two digit values, 20 and 21 means we want to orthogonalize against some other states in the basis,
the first digit (2 in our case) is the energy set which contains the states we want to orthogonalize against and the second digit is the l-quantum
number in that energy set to orthogonalize against. So, in our setup the 3s and 3p states (in the second energy set), have linearization energies
determined by the center of their bands. The 3d states (in energy set 1) also uses the center of band method. The 4s and 4p however will be
orthogonalized against the 3s and 3p respectively (20 means orthogonalize against the s-states in energy set 2, similar for 21).

We will come back to the choice of linearization flags later.

## Fourier grid
```
 (/ 3i6, 4x, 2i1, 2i6)
   ft1   ft2   ft3    ..  diag     .
    20    20    20    00     2     0
```
RSPt uses the Fast Fourier Transform (FFT) as part of the full potential scheme.
The FFT needs a grid of points to sample, that grid is set in this file. Usually
the default settings are fine, but sometimes we need a finer grid. We will talk more about the FFT grid later.

## Header
```
 (/ 2i6, f6.0, 2i6, 2f6.0, 4l6, i6)
  lmax ntype  zval     . icorr     .  pmix   win   wmt f-rel sp-po     .
    12     1  18.0     8    34    1.  .500     t     t     f     f     1
```
The header file contains a bunch of settings for the DFT calculation, the important ones for now are
`icorr`, which determines the exchange-correlation functional to use, and `pmix` which sets the linear mixing parameter.

icorr is not a single number, rather it is 2 integers in a row, by defauilt 3 and 4. The XC functional chose is determined by what combination we select.
34 means the AM05 GGA functional, 24 means PBE96, 44 PBESol. If we want to use an LDA functional we set the first digit to 0,
04 means the Perdew-Wang LDA parametrization. 02 is for the extremely patriotic Swedish LDA lovers.
Usually 34 is a good choice for GGA, and 04 is a good choice of LDA. Leave it at 34 for now.

pmix sets the linear mixing parameter used in the early steps of the Broyden mixing scheme,
if you have a complicated system you probably need to decrease this, at least by an order of magnitude.
For our case, 0.500 is good, so we leave it as is.

sp-pol determines whether we have a spin-polarized calculation or not, however, changing the sp-pol flag is
not recommended, because other input files need to be modified so that they match whatever sp-pol is. Therefore it is usually
best to set the `symt.inp` to do a spin-polarized calculation and rerun `symt -all`, more on this later.

f-rel is an abbreviation of "fully relativitic", which is RSPt for "spin-orbit coupling". If you want to include spin-orbit coupling this flag should be t (which is Fortran for true).
Changing this flag might require changing other input files, so it might be best to redo the `symt.inp` setup, this tutorial won't cover spin-orbit coupling.

## prnt array
```
    .    1    .    2    .    3    .    4    .    5    .    6    .    7
fftffffftffffffffffftftfffffffftfffftffffffftffffffffffffffftfffffttffff
```

The prnt_array is one of the main selling features of RSPt. It contains an array of 72 boolean (true/false) values.
These 72 booleans control different parts of the RSPt calculation. Usually we don't want to change this thing, but sometimes we have to. More on this later...

The numbers on the first row indicate the 10s of the indices, each dot is a 5 (first dot is above prnt_array(5), the one is above prnt_array(10), and so on).
This is purely to help the user, true user-friendly design right there.

(some of these flags are not used at all others should NEVER be changed)

## Tail energies
```
 (f12.0, f5.0, i1, i3, 3i2, l3, f6.0, i3, 6f12.0)
         0.3   .01  0 1 0 1  f    .0  0          .0
        -2.3   .01  0 1 0 1  f    .0  0          .0
        -1.5   .01  0 1 0 1  f    .0  0          .0
```
The tail_energies file contains the kinetic energy tails that we want to use in our basis (remember the 3rd column of integers in the LMTO setup in element?).
This basically describes how our basis functions behave outside of the MT. A negative value means they are exponentially decaying, larger magnitude => faster decay rate.
A positive value means they are oscillating. Positive values can be dangerous when using dense k-point meshes, especially for DMFT, and therefore the default choice of the first tail energy should be changed.
Change the first tail energy to -0.3, do this by `replacing` the space in front of the 0 with a - sign, do not shift any of the other values!

With these changes we are now ready to prepare the main input file for a RSPt calculation. First we need to copy the files `length_scale` and `strain_matrix` from the dta folder
to the main calculation folder (because of reasons...)

Then, in the main calculation folder do `make data`, to create the data file. This is the actual input file that RSPt reads. Our work up to this point
has been to prepare files that set parts of the data file. Usually it is easier to change the smaller files in the dta folder and then `make data` to update the data file, but
brave people can make changes in data directly if they want to.


# Running RSPt
By launching the executable `rspt` we run iteration of the DFT Self Consistent Field (SCF) cycle.
After each iteration we check if the caclulation is converged or not, if not converged we run another iteration, If
the calculation is converged we stop.

RSPt puts all the output in the `out` file.

## Checking for convergence
The main quantity to check for convergence is the `fsq`, this measures how much the newly generated electron densities and potentials
vary from the input ones. fsq is printed at the very end of the out file. Generally, you need fsq below 1e-12 for a calculation
to be converged. But, always check that the quantity you are interested in has stopped changing (total energy, magnetic moments,
density of states, etc.), sometimes an fsq of 1e-10 is enough, sometimes you need 1e-15 or even smaller.

The file `hist` contains a summary of each SCF iteration done so far. It can be useful to check how things change
between the SCF iterations.

## Checking for signs of trouble
Many things can go wrong in a RSPt calculation, and the code could happily converge to somethign completely insane.
Therefore we need to check the early iterations for signs of trouble. In the out file, many things are printed, we will look at a few here.

### Overlapping muffin tins
```
 Nearest neighbors: min_{b+R} |b+R-a|, |b+R-a| < 1.1 (min|R|= 4.6436225)
  ta  tb         d/L        2S/d   d(au)    d(A)    a    b
   1   1 .7071067811 .9501596214 4.64362 2.45729    1    1
```
The 2S/d column shows how close together the MT of various atoms are. A value of 1 or more means they overlap, and RSPt crashes.
We fix this by decreasing the MT radii of the overlapping spheres (remember the first lines of the element file?). If the value
if below 0.90 for nearest neighbors, it might be a good idea to increase the MT radii instead. We want to keep the MT large, but not too large.
A 2S/d of more than 0.98 is also potentially bad, but the code probably won't crash. Aim for a value of 0.95-0.96.
(For DMFT 0.98 might be preferable, sometimes, maybe)


### Energy parameters
In the output file, there is a section called "Energy parameters"
```
      Energy parameters ...                          energy reset
                                                     | npr(mt) .ne. npr
                                                     | | e < e1(ws)
        t  e  l        n mt           E       D(mt)  | | | e > e2
        1  1  0        4  4   2.4563842      -6.684  * - - *
              1        4  4   3.2893295      -5.467  * - - *
              2        3  3   0.5993703      -2.142  - - - *
              3        4  4   0.5966323       2.392  - - * -
              4        5  5   0.6231592       3.552  - - * -
              5        6  6   0.6605288       4.630  - - * -
              6        7  7   0.6513481       5.695  - - * -
              7        8  8   0.6501201       6.739  - - * -
              8        9  9   0.6274272       7.777  - - * -
           2  0        3  3  -6.7774311      -6.683  * - - -
              1        3  3  -3.8996796      -5.467  * - - -
              2        3  3   0.2557125       0.269  - - * -
              3        4  4   0.3662351       2.537  - - * -
              4        5  5  -0.0014851       3.847  - - * -
              5        6  6  -0.7907065       5.185  - - * -
              6        7  7  -1.5471901       6.408  - - * -
              7        8  8  -1.8372524       7.447  - - * -
              8        9  9  -1.6120157       8.349  - - * -
```
The most important part is the `npr(mt) .ne. npr` column, this is a check to see
if our LMTO basis functions have the correct number of nodes inside the MT or not.
If there is a '*' anywhere in this column your calculation might be in trouble.
To get rid of the '*' it is usually enough to change the linearization flag in the element file.
The flag is set for a specific l-quantum number, for a specific energy set, for a specific type.
Check what element the type is, and change the linearization flag for the l-quantum number in the energy set e to -1 in the correct element file.


### Fourier transform parameters
```
 Fourier transform parameters
 radii and arguments:
 2.2060912     6
```
See that 6? any number smaller than 6 is bad. 6 or larger is good. To get the value to 6 or above,
increase the number of points in your FFT mesh (I usually round up to the nearest power of 2).

### Charge densities at boundaries
```
 Charge densities at boundaries
     t     h           n
     1     1  .047285309 ... mt
              .047284139 ... int
           2 -.006547523 ... mt
             -.006563025 ... int
           3 -.005002146 ... mt
             -.005191644 ... int
           4  .000696130 ... mt
              .001018920 ... int
           5  .000052774 ... mt
              .000044586 ... int
           6  .000010342 ... mt
             -.000021512 ... int
           7 -.000021782 ... mt
             -.000155012 ... int
```
This shows the electron densities at the boundary between the muffin tins and the interstitial.
Make sure that for the first 3 (or 4) harmonics (h), the difference is below 1e-3 or 1e-4.
If the differences are too large, increase the number of points in the FFT mesh.

### L-projected averages
```
 L-projected averages ...
      Muffin tins
        t  l     e                    E           dE/E              Q
        1  0     1            0.3350207      0.0000072      0.3430976
                 2           -6.5397976      0.0000008      2.0684969
             total           -5.5617169                     2.4115945
           1     1            0.5017692      0.0000070      0.2433014
                 2           -3.7871534      0.0000009      6.1466433
             total           -3.6238498                     6.3899447
           2     1            0.5993671     -0.0000053      8.0431877
                 2            0.2557179      0.0000210      0.0471624
             total            0.5973638                     8.0903501
           3     1            0.5966266     -0.0000096      0.0205687
                 2            0.3662388      0.0000101      0.0035661
             total            0.5625853                     0.0241348
           4     1            0.6231527     -0.0000104      0.0047226
                 2           -0.0014705     -0.0099293      0.0008242
             total            0.5303402                     0.0055468
           5     1            0.6605228     -0.0000090      0.0012848
                 2           -0.7906923     -0.0000180      0.0001528
             total            0.5062668                     0.0014377
           6     1            0.6513436     -0.0000069      0.0003038
                 2           -1.5471906      0.0000003      0.0000367
             total            0.4142064                     0.0003405
           7     1            0.6501161     -0.0000062      0.0000752
                 2           -1.8372656      0.0000072      0.0000104
             total            0.3483514                     0.0000856
           8     1            0.6274257     -0.0000023      0.0000183
                 2           -1.6120334      0.0000110      0.0000034
             total            0.2793171                     0.0000217
```
This section shows the occupation (Q), projected onto out LMTO basis functions. Make sure
that the values are reasonable. In our case we have Ni atoms, so the 2s (s-orbitals in energy set 2) should be fully occupied. The same goes for 2p.
The 3d (d-orbitals in energy set 1) should have 8-9 electrons. The values won't match your atomic intuition perfectly
just make sure that you are not 2-3 (or more!) electrons away. If something goes wrong, make sure that you check the
Energy reset parameters, Fourier transform parameters, and the matching at sphere boundaries.

If your problem remains, after everything else seems fine, try reducing the mixing parameter and restart the calculations from atomic densities.


### Total charges
```
 Valence Charge Summary
        type                       Q
           1         16.923456407448
           I         1.0765435925508
```
Make sure that the valence charge of each type is reasonable (similar criteria as for L-proj), make sure that the valence charge is close
to what you expect, and the interstitial is not too large. If not, check the flags, check the mixing, and finally if all else fails, change the MT radii.

## When do things normally go wrong?
Usually, iteration 1 looks very good (always check!), because it will be just a small step away from the atomic configuration we started with.
The Energy parameters will start showing problems from the second iteration. Check the first few (2-3) iterations carefylly! The mixing method of RSPt
(Broyden mixing) does linear mixing for the first few (I think about 7) iterations, then it switches to the Broyden scheme. If you are unlucky, sometimes
this switch is rather aggressive, and things go wrong. Therefore check iterations 7-10 for problems as well. If you have problems when broyden mixing turns on
you can try reducing the mixing ratio, or you can erase the files used by the Broyden scheme, jacob1 and jacob2, in order to force RSPt to continue with
linear mixing for a few more iterations (there might be a prnt`array flag to do exactly this, if you are brave/stupid).

So, in summary, the first 10 or so iterations are usually where things go wrong. So be vigilant in the beginning.

## FIles needed to restart or continue a run
RSPt always needs the files `data`, `spts`, `tetra` (if doing tetrahedron method), `symcof`. If the files `pot` and `eparm` are available, RSPt will
continue the SCF cycle from them. If `pot` and `eparm` are absent, RSPt will try to restart from the atomic densities from the file `atomdens`.

You can completely restart a RSPt run by deleting the files `pot`, `eparm`, `jacob1`, and `jacob2`.

# The magic of runs
```
runs "rspt" 1e-15 200
```
Since `rspt` only runs a single SCF iteration, and does not check convergence at all, there is a utility program called `runs`.
This program takes the command used to run a single SCF iteration (usually "rspt", or "mpirun -n $SLURM_NTASKS rspt", ..., use the double quotes!!!), the fsq convergence limit you want
as well as the maximum number of SCF iterations to run. runs will then run RSPt for you, until converged, maximum number of iterations reached, or sometimes crashed. runs will eat
any error messages produced when running RSPt and put them in the file  `runs_errfile`. runs will also backup some files from the last SCF iteration by copying them and adding the "_last" suffix
(usually pot_last, eparm_last, and out_last).

runs is a very simplistic tool, but it does automate the SCF iterations and allows for some basic convergence check. It does not feature any type of error detection, so use it only when you are
certain that your settings are reliable. runs also modifies the hist file when it ends the SCF run, only saving the last SCF run performed. I personally strongly dislike this.


# Task
Please converge a FCC Ni calculation. Starting from scratch, follow the steps outlined, and do a fully converged SCF cycle. Don't worry about converging the k-point mesh, just use the coarse grid we
specified in the early steps. Ask for help if anything goes wrong!

# The manual
RSPt comes with a manual, in the documentation folder. It contains a lot more detailed information, some of it is even correct! Don't be afraid of the manual,
but also, don't trust the manual. Sometimes it is out of date. The manual is for experienced users, and brave beginners.

## The code
If you are very brave (or stupid, or a developer) you can of course check the RSPt source code and try to figure out what's going on. But please note, the code is not
written to be easily read by humans... Some parts are quite readable (the DMFT part usually), others (the baisc functionality regarding the LMTO basis comes to mind) are definitely not.
