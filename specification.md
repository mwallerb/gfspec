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

 * Key: A prefix that determines the type of child (not part of the name): @ denotes HDF5 attributes, + denotes HDF5 sub-groups, and no prefix is used for HDF5 datasets. Followed by the name of the child in the HDF5 file.

 * Data type: one of int, string, or double.  Strings are encoded as fixed-length character arrays.  May optionally be followed by array shape in square brackets in the following form: `[dim1]`, `[dim1,dim2]`, etc; `[...]` denotes an arbitrary multi-dimensional array. May optionally be followed by an equal sign and the set of allowed values.

 * Presence: whether the argument is required
 
 * Description: short description.

Any implementation *must* understand every child specified here, and *must* ignore any additional children present it does not understand.  Implementations-defined children *may* be added, and the names of such children *should* begin with an underscore. Implementations *should* issue a warning for any unknown fields that do not begin with an underscore.


Green's function group
----------------------
The topmost object of any Green's function object is characterized by a Green's function group, which corresponds to a HDF5 group with `@kind='GF'`. The group may have any name and may appear anywhere in the group hierarchy of a HDF5 file (i.e., it need not coincide with the root of the hdf5 file).  Multiple Green's function object may be present in the HDF5 file, but they must not be directly or indirectly nested.

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


Dataset `data`
--------------

`data` is a multi-dimensional dataset of double values.  Its children are:

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`__complex__`      | int = 1           | iff data is complex | Tag for complex-valued data |

The attribute `__complex__` must be present if and only if the data is complex-valued.  In case of complex-valued Green's functions, the numbers are stored as real in one more dimension, with the last (fastest) dimension of size 2 discriminating between real part (index 0) and imaginary part (index 1).

The number of dimension of the data is also stored in `mesh/N` (see below).  The ordering of the data is such that the first (slowest changing) index belongs to the first mesh (see mesh section), and the last (fastest changing) index belongs to the last mesh.

Subgroup `mesh`
---------------
`mesh` contains the grids/meshes on which the Green's function is stored.  There is one mesh per dimension of the Green's function data, stored at `/mesh/1`, `/mesh/2`, etc.  The dimensions of the data must correspond to the number N of meshes; mesh `<n>` corresponds to the `n`-th dimension of the `data` dataset.

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string = <br/> `'CartesianProductMesh'` | required | Identification tag |
| +`1`                |                   | required | Mesh for the first dimension |
| +`2`                |                   |          | Mesh for the second dimension |
| +`3`                |                   |          | Mesh for the third dimension |
| ...                 |                   |          | ... |

Each mesh has the following children:

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string            | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |
| ...                 |                   |          | Further domain-specific datasets ... |

A list of mesh types follows:

**Index mesh**

Index meshes describe simple indices (like spin or orbital indices):

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'MeshIndex' | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |


**Matsubara frequency mesh**

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'MeshImaginaryFrequency' | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |
| `beta`              | double            | required | Inverse temperature |
| `statistics`        | string = 'B','F'  | required | `B`osonic or `F`ermionic statistics |
| `positive_freq_only`| int = 0,1         | required | Only positive frequency axis stored? |
| `points`            | double[`size`]    |          | explicit values of the Matsubara frequencies |

This defines the grid $ \omega_n = (2n+1)\pi/\beta $ for fermions, $ \Omega_n = 2n\pi/\beta $ for bosons.

For fermions: $n=0..(N-1)$ (N grid points) if only positive frequencies are stored; $n=-N, -(N-1), .., -1, 0,..(N-1)$ (2N grid points) if both frequencies are stored. 

For bosons:  $n=0..(N-1)$ (N grid points) if only positive frequencies are stored; $n=-(N-1), .., 0,..(N-1)$ (2N-1 grid points) if both frequencies are stored.

If the optional parameter `points` is specified, it *must* be verified upon reading.

**Imaginary time mesh**

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'MeshImaginaryTime' | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |
| `beta`              | double            | required | Inverse temperature |
| `statistics`        | string = 'B','F'  | required | `B`osonic or `F`ermionic statistics |
| `last_point_included`| int = 0,1        | required | 0 if the last point is at $\beta$, 1 otherwise |
| `half_point_mesh`   | int = 0,1         | required | 0 if points are shifted by $\beta/N*0.5$, 1 otherwise |
| `points`            | double[`size`]    |          | explicit values of the Matsubara frequencies |

**Linear real frequency mesh**

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'MeshRealFrequencyLinear' | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |
| `min`               | double            | required | Frequency of the lowest meshpoint |
| `max`               | double            | required | Frequency of the highest meshpoint |
| `points`            | double[`size`]    |          | explicit values of the real frequencies |

**Arbitrary real frequency mesh**

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'MeshRealFrequency' | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |
| `points`            | double[`size`]    |          | explicit values of the real frequencies |

**Legendre mesh**

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'MeshImaginaryTime' | required | Discriminates between mesh types/domains |
| `label`             | string            |          | An optional string labeling the mesh instance |
| `size`              | int               | required | Number of mesh points/data points along dimension |
| `beta`              | double            | required | Inverse temperature |
| `statistics`        | string = 'B','F'  | required | `B`osonic or `F`ermionic statistics |

TODO: real space/momentum space

Subgroup `tail`
---------------
The tail group contains a model of the Green's function, mostly to take care of the behavior at infinite frequency.  For a simple power expansion (`TailGFPower`), it is written as:

G(i\omega_n) = c_0 + c_1/(i\omega_n) + c_2/(i\omega_n)^2+...

High frequency tails are only defined if there is only one Matsubara/imaginary time/ real time/ real frequency axis. They are not defined for multiple-frequency vertex functions.

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| @`kind`             | string =<br/>'TailGFPower' | required | Discriminates between mesh types/domains |
| `min_tail_order`    | int               | required | Lowest power of $1/iw$ stored |
| `max_tail_order`    | int               | required | Highest power of $1/iw$ stored |
| `data`              | double[n,...]     | required | Tail data |

The minimum and maximum high frequency expansion coefficient indices are stored in min_tail_order and max_tail_order. min_tail_order=0 corresponds to a constant term, min_tail_order=1 to the 1/(i omega_n) term, etc.

The first dimension of data is `max_tail_order - min_tail_order + 1`, and the tails are stored in order of ascending powers along that direction.  The remaining dimensions *must* correspond exactly to the non-frequency dimensions of the Green's function data group (see there).

For Green's functions which are not stored in Matsubara frequencies, these coefficients describe the high frequency tails of the function transformed to Matsubara frequencies.

Subgroup `version`
------------------
Version of the hdf5 specification this data file adheres to, with minor and major version. Current `major` version is 3, current `minor` version is 0. `reference` contains a string pointing to the URL of this document. `originator` is a program specific string that describes the program that wrote this file.

| key                 | data type         | presence | Description |
|---------------------|-------------------|----------|-------------|
| `major`             | int               | required | Version number, major digit |
| `minor`             | int               | required | Version number, minor digit |
| `reference`         | string            | required | URL of a copy of this document |
| `originator`        | string            | required | String specifying the implementation |

