# Compressed Sparse Fiber Matrix

CSF Matrix is a read-only sparse matrix class optimized for sparse-dense computation in cases where non-zero values are highly redundant. For such cases, sparse fiber storage can reduce memory footprint by up to 50% compared to standard sparse compression. CSF also increases the ability to further compress index arrays within each fiber.

## What is CSF?

Consider a dense vector:

```
x = {1, 0, 0, 0, 2, 0, 1, 0, 0, 2, 1, 1, 0, 1};
```

This can be stored in CSC format:

```
x = {1, 2, 1, 2,  1,  1,  1};
i = {0, 4, 6, 9, 10, 11, 13};
```

However, the values are highly redundant and so we can adapt "run-length encoding" for paired vectors. We can order `x` and `i` first by value in `x` and second by value in `i`, then store only unique values in `x` and the runs of these unique values in `j`:

```
x = {1, 2}
j = {5, 2};
i = {0, 6, 10, 11, 13, 4, 9};
```

In vector form:

```
v = {1, 5, 0, 6, 10, 11, 13, 2, 2, 4, 9};
//  /   \  |---------------| |   \ |---|
// value run   indices    value run indices
```

Alternatively, store delimiters (`0`) between runs instead of run lengths:

```
v = {1, 0, 6, 10, 11, 13, 0, 2, 4, 9};
//   |  |--------------|  /   \ |---|
// value    indices  delimiter value indices
```

We refer to each index run (+ delimiter if applicable) as a **fiber**.

## Random Access is Unordered

Unlike CSC, where values in any column are ordered by row index, CSF values are ordered firstly by value and secondly by row index. Thus, sparse-sparse operations become very inefficient, while sparse-dense operations remain relatively efficient. Thus, CSF is useful for two cases:

* Low memory footprint
* Fast sparse-dense operations

## C++ API

The `CSFmatrix::SparseMatrix<T_value, T_index, compression_level>` class feels a lot like an `Eigen::SparseMatrix`. There are three compression levels:
 1. Column Sparse Compressed (CSC) format
 2. Column Sparse Fiber (CSF) format
 3. CSF + positive-delta encoding and bytepacking of indices in fibers
 
Use compression level 1 if performing many sparse-sparse column-wise operations or if non-zero values are not highly redundant. Use compression level 3 if slightly compromised random access speed is acceptable for another significant reduction in memory footprint.

## R API

The R API provides three S4 classes, one for each compression level, that act a lot like `Matrix::dgCMatrix`: `cscMatrix`, `csfMatrix`, and `csfMatrix2`.

These classes provide zero-copy access to the C++ header library using Rcpp data structures and support basic matrix operations.
