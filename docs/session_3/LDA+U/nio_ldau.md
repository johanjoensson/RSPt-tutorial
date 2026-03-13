# Setup
[AFM NiO](../../session_1/NiO_AFM/nio_afm.md)

Use the same setup as before.

# Converge SCF
([see session 1 for more info](../../session_1/FCC_Ni/fcc_Ni.md#Running-RSPt))

Do a rough convergence of the SCF cycle (maybe 10 iterations, or an fsq of 1e-6). This weill be our starting point for the LDA+U calculations.

# LDA+U setup
RSPt uses the Greens function machinery to do LDA+U, so we will set up a `green.inp` file for that exact purpose.
```
inputoutput
 T T  # read_sig csc

convergency
 1e-6 1e-4 999 99  # delta_n delta_sig max_it max_solver_it

cluster
 1 UJ eV IdNiUp_d  # norb U? Identifier
 1 2 1 1 3 8.0 1.0  # t l e site basis U J
 2 2 0.65           # solver DC mixing

cluster
 1 UJ eV IdNiDn_d
 2 2 1 1 3 8.0   # t l e site basis U
 2 2 0.65
```
Some changes from before! We are not calculating DOS/pDOS right now, so no spectrum block. We are doing
a self consistent calculation, where we calculate a new self-energy to update the KS-DFT electron density,
the `inputoutput` block contains 2 boolean values, the first one controls if we
are going to read any previously calculated self-energy (called `sig`) or not, and whether or not we
want to produce a new electron density or not (for a self-consistent calculation this is required).

The `convergency` block contains limits for convergency that we will use in the LDA+U/DMFT cycle.
`delta_n` determines the accuracy in number of electrons we want to use for finding the Fermi energy
(when we start putting in self-energies the Fermi energy will start shifting, and so we need to update it for every
new self-energy). `delta_sig` is the convergence cutoff for the calculated self-energies. For LDA+U the self-energy
should converge nicely, and 1e-4 is a rather easy target. For other solvers a larger `delta_sig` is probably needed.
`max_it` is a maximum number of times the Fermi energy will be updated before we take a break and do some DFT stuff.
`max_solver_it` is the maximum number of times we update the self-energy before the aforementioned DFT stuff break.
The iteration numbers are not that important, but setting `max_it` to 999 and `max_solver_it` to 99 is usually good.

The clusters now have some extra info. No 0 on the first line any more, because we want a Coulomb U!
`UJ` means that we will specify the (Hubbard) U and (optionally) J parameters to use, `eV` means  that the
parameters are given in eV instead of Ry. If we don't know what value to use for J, we can let RSPt make an
educated estimation and try to calculate a J for us. To do this we just give a U value, no J. If we don't specify
the `UJ` flag, RSPt will expect us to provide the Slater integrals to use. If we only know the first nonzero Slater
integral (the Hubbard U), we can just skip the others and RSPt will estimate them for us. The estimation is useful,
but quite often we will want to specify exactly the value of J we want.

The final, third line in the cluster blocks contains information about how to solve the LDA+U/DMFT problem.
The solver flag selects the solver to use, 2 means LDA+U, the DC flag selects the double counting to use, 2 means fully
localized limit (FLL), and mixing is the mixing parameter to use for the self-energy mixing.

The manual has a very extensive section talking about the `green.inp` settings, have a look there.

# Run SCF iterations
Now, for each SCF cycle, RSPt will converge the LDA+U self-energy and then break. The self-energy is saved in the binary file `sig`. Unfortunately
`runs` does not save the `sig` file after each iteration, so be careful not to delete it by mistake. The file `dmft_hist`
contains a summary of the self-energy self-consistency cycle (similar to `hist` for the DFT SCF), it lists the change in
self-energy from the last solver iteration, the current estimate for Fermi energy (or, chemical potential),
the number of electrons obtained with the current Fermi energy, and some more stuff.

## Plot dos
After converging the LDA+U SCF cycles, modify the `green.inp` file to calculate the DOS/pDOS, and plot it.
What has happened to the states near the Fermi energy (in particular the states we put a U on).
