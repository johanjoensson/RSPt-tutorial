# Installing RSPt
To install RSPt we use the tool Make, which is old school.

Settings (libraries, compilers, etc.) are set in the file `RSPTmake.inc`,
change these to fit your system. If you don't know what this is, don't worry.
If you want to look at a few different configurations, take a look in the `RSPTmakes` folder.
Inside you will find a number of files containing settings for supercomputer clusters with different
build chains. Again, if you don't understand what this means, don't worry.

## Test to see if you need to install RSPt
There is a precompiled binary in the `bin` folder, maybe that will work on your computer.
To test the binary, first modify the file `testsuite/runcommand` (if that file does not exists instead modify
`testsuite/runcommand.default`)
```bash
#!/usr/bin/env bash
MPIRUN=""
EXEC="rspt"
```
This file contains commands used to run RSPt on your machine. `EXEC` is the name of the RSPt binary you
want to use (in our case, and most cases, it is `rspt`), `MPIRUN` is the command used to launch a binary with
MPI. In this tutorial we will not use MPI and therefore `MPIRUN` is left empty. For serious calculations it is basically a must
to use MPI parallelization, otherwise calculations will run too slowly. In the final session we will talk a bit about MPI
parallelization.

To run the testsuite, type `make test`. All tests should pass.

## Install RSPt
If you need to build RSPt, and have set up the `RSPTmake.inc` the way you want, just type `make pristine`, this will remove any precompiled things
in your rspt directory. Then just type `make`, and the code will compile the binary `rspt` (and a few more). Binaries compiled are stored in the folder `bin`

## Add binaries to your PATH
If you want to add the RSPt binaries to the runtime path (which you probably do), use the
command (or something that achieves the same result)
```bash
export PATH="$PWD/bin:$PATH"
```
This adds the folder `bin` to your runtime path.

Feel free to run the testsuite to verify that your newly compiled RSPt binary actually works, see [above](##Test-to-see-if-you-need-to-install-RSPt).
