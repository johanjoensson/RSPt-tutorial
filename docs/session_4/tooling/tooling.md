# Tools for using RSPt
We are already familiar with the `runs` program, this is one of
a family of utility programs built when building RSPt. We won't talk
about the other programs in the "runs" family, check the manual. However, there
are some interesting tools available that can help with your RSPt calculations.

- [cif2cell](https://pypi.org/project/cif2cell/), a program that can create symt.inp files for you, starting from a CIF file.
- [ASE](https://ase-lib.org/index.html) Python modules useful for setting up and modifying crystals. Not RSPt specific, but can produce CIF files (see above for why that is useful).
- [gnuplot](http://www.gnuplot.info/) If you want to plot something from RSPt, usually it is easy to do so using gnuplot.
- [Grace](https://plasma-gate.weizmann.ac.il/Grace/) If gnuplot is too modern and not graphical enough for you, then maybe Grace is what you want.
- [pyRSPthon](https://github.com/johanjoensson/pyRSPthon) My own python modules for many different things that I have found useful for RSPt calculations. Under heavy development, changing often, not much documentation available right now.
