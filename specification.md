Simons correlation function interchange format
==============================================

Scope
-----
This specification defines a data format for storing correlation functions, with the aim portability and interoperability across different codes, algorithms, and research groups.

To minimize breakage, the specification tries to follow two main influences as closely as possible: the ALPSCore H5GF specification and the relevant subset of the triqs file format.


File format
-----------
Green's functions are structures are stored in the HDF5 file format.

Each Green’s function corresponds to a hierarchy of specifically named nodes (groups, datasets and attributes) within a file.  Any implementation *must* understand every node specified here, and *must* ignore any additional nodes present it does not understand.  Implementations may add additional nodes not specified here: the names of such nodes *should* begin an underscore.  Implementations *should* issue a warning for any unknown fields that do not begin with an underscore.

Quantities and data outside the hierarchy for Green's functions, which may be present in the HDF5 file, are outside of the scope of this specification.


Green's function group
---------------------
Each Green's function object is characterized by a Green's function group, which corresponds to a HDF5 group. The group may have any name and may appear anywhere in the group hierarchy of a HDF5 file (i.e., it need not coincide with the root of the hdf5 file).  Multiple Green's function object may be present in the HDF5 file.  Green's function object may not be directly or indirectly nested.

Each Green's function group shall have the following structure:

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string = `'GF'`   | required | Identification tag |
| @`target_space_dim` | int               |          | Dimension of the target space (0=scalar, 2=matrix, ...) |
| `data`              | double[...]       | required | Data as multi-dimensional array |
| +`tail`             |                   |          | Asymptotic tail of the function |
| +`mesh`             |                   | required | Metadata associated with the array dimensions |
| +`version`          |                   | required | Version information |




Notation
--------









x
