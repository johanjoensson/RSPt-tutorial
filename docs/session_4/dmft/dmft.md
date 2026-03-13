# DMFT in RSPt
For doing DMFT in RSPt, we need a bit more than just LDA+U. LDA+U can
be thought of as a really very simple DMFT solver. For more advanced solver methods
we need a bit more details and settings.

```
inputoutput
 t t

matsubara
 2048 12 120 8  # n_mats  n_head  n_body  n_tail

energymesh
 2001 -1.00 0.5 0.015

convergency
 1e-6 1e-3 999 99

mixing
 1 0.15 0  # mix_method  max_slope  sigdiff_factor

projection
 2        # projection method

tensmom
 Symbrk
 0  1  1  1.0

cluster
 1 IdNiDn
 2 2 1 1 3 0.5512 0.7276 0.4851 Ewin -0.8 0.2
 6 1 0.45 0.10
 8 0 2 2 3

cluster
 1 IdNiUp
 1 2 1 1 3 0.5512 0.7276 0.4851 Ewin -0.8 0.2
 6 1 0.45 -0.10
 8 0 2 2 3
```
For an explanation of the `tensmom` block, see [here](../non_magnetic_dft_magnetism/symbrk.md).

The `matsubara` block sets up the Matsubara frequency mesh we will use. `n_mats` is the number of Matsubara frequencies in our mesh. This
needs to be large enough for our problem, and the number of frequencies required is both temperature and system dependent. Usually 1000-2000 frequencies
are required, to be safe. `n_head`, `n_body`, and `n_tail` are a bit special. RSPt will use a linear-logarithmic-linear mesh to interpolate the Matsubara
frequencies, for performance reasons. `n_head` sets the number of frequencies to keep in the first linear part of the mesh (20 would be very high). `n_body`
sets the number of logarithmic mesh points to use (a couple hundred is usually more than enough). `n_tail` sets the number of linear mesh points to
use at the very end of the Matsubara mesh (again, 20 would be a lot). Check the manual for more info on selecting the Matsubara mesh.

The mixing block is special! For DMFT we do a self-consisten cycle for the self-energy, mixing determines how we mix the self-energy.
The first integer determines the method to use for mixing, 1 means linear mixing, 3 is a weird but efficient thing with Broyden mixing.
If you struggle to converge the self-energy, linear mixing with a mixing paramenter of ~(0.45, 0.65) could be worth trying.
The other numbers you can usually just keep as they are. See the manual for more info.

In DMFT we need localized orbitals to put a U on. We get these orbitals via a  prjection
scheme. The `projection` block let's us select that scheme. 1 means a scheme based only on the LMTO basis and the
parts of basis functions inside the muffing tins. Usually not very complicated, and maybe not that good.
2 Is a more involved scheme including tails etc. from other muffin tins, usually a bit more tricky to set up
because it (_almost_) always requires appropriate energy windows for setting up the projectors. Usually better
than option 1, maybe.

In the `cluster` blocks we no see the appearance of `Ewin`, which let's us select energy windows
for our localized orbital projectors. Check the 'pdos' files and `hybridization` (spectrum flag `Hyb`)
for the DFT calculation and select an energy window that is large enough to include all the states you want, and
all their hybridization, but no states that you don't want (and their hybridization). As an example, in NiO, make
sure that your energy window captures all the Ni d-states, but not the O p-states. Choosing as large an energy window
as possible, without including too much stuff, is a good idea.

In this example the clusters are set up to use the exact diagonalization solver for the DMFT problem, check the manual for more info.
