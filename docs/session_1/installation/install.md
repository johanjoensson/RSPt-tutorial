# Installing RSPt
To install RSPt we use the tool Make, which is old school.

Settings (libraries, compilers, etc.) are set in the file `RSPTmake.inc`,
change these to fit your system. If you don't know what this is, don't worry.
If you want to look at a few different configurations, take a look in the `RSPTmakes` folder.
Inside you will find a number of files containing settings for supercomputer clusters with different
build chains. Again, if you don't understand what this means, don't worry.

## Install RSPt
If you need to build RSPt, and have set up the `RSPTmake.inc` the way you want, just type `make`, and the code
will compile the binary `rspt` and a few more. They are stored in the folder `bin`

## Add binaries to your PATH
If you want to add the RSPt binaries to the runtime path (which you probably do), use the
command
```bash
export PATH="$PWD/bin:$PATH"
```
This adds the folder `bin` to your runtime path.

## Running tests
You can try running the RSPt test suite,
```bash
make test
```
None of the tests should fail.
