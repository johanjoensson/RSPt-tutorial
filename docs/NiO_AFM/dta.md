# Settings
This time, sp-pol should be true in header, if it is not you need to add
spinpol to the _symt.inp_ file. Also, the linearization flags block for each
element now contains double the number of rows. Still they are divided into
the different energy set, now 4 rows per energy set. Each energy set is then
further divided into the two different spin projections, spin down and then spin
up.

Negative tail energies!

Copy _length_scale_ and _stran_matrix_. Make the data file.
