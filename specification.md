Simons correlation function interchange format
==============================================

Scope
-----
This specification defines a data format for storing correlation functions, with the aim portability and interoperability across different codes, algorithms, and research groups.

To minimize breakage, the specification tries to follow two main influences as closely as possible: the ALPSCore H5GF specification and the relevant subset of the triqs file format.


File format and notation
------------------------
Green's functions are structures are stored in the HDF5 file format. Quantities and data outside the hierarchy for Green's functions, which may be present in the HDF5 file, are outside of the scope of this specification.

Each Green’s function corresponds to a hierarchy of specifically named HDF5 groups.  The children of each such group are specified in a table with the following columns:

 * Key: name of the child. The prefix determines the type of child: @ denotes HDF5 attributes, + denotes HDF5 sub-groups, and no prefix is used for HDF5 datasets.

 * Data type: one of int, string, or double.  Strings are encoded as fixed-length character arrays.  May optionally be followed by array shape in square brackets in the following form: `[dim1]`, `[dim1,dim2]`, etc; `[...]` denotes an arbitrary multi-dimensional array. May optionally be followed by an equal sign and the set of allowed values.

 * Presence: whether the argument is required
 
 * Description: short description.

Any implementation *must* understand every child specified here, and *must* ignore any additional children present it does not understand.  Implementations-defined children *may* be added, and the names of such children *should* begin with an underscore. Implementations *should* issue a warning for any unknown fields that do not begin with an underscore.


Green's function group
---------------------
Each Green's function object is characterized by a Green's function group, which corresponds to a HDF5 group. The group may have any name and may appear anywhere in the group hierarchy of a HDF5 file (i.e., it need not coincide with the root of the hdf5 file).  Multiple Green's function object may be present in the HDF5 file.  Green's function object may not be directly or indirectly nested.

Each Green's function group shall have the following structure:

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string = `'GF'`   | required | Identification tag |
| @`target_space_dim` | int               |          | Dimension of the target space |
| `data`              | double[...]       | required | Data as multi-dimensional array |
| +`mesh`             |                   | required | Metadata associated with the data array dimensions |
| +`tail`             |                   |          | Asymptotic tail of the function |
| +`version`          |                   | required | Version information |

The array shape of `data` must be consistent with the `mesh` stored.

If specified, `target_space_dim` denotes the dimension of the target space of the Green's function (0 meaning scalar, 2 meaning matrix, etc). It *must* be an integer between 0 and the number of dimensions of `data`, and `data` *must* be square in the last `target_space_dim`'th dimensions.


--------









x
