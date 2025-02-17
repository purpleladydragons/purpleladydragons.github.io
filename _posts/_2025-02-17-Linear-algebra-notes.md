Going through 3Blue1Brown's series as a refresher

Intuition around matrix multiplication
- relationship between any vector and basis vectors is maintained
- so if you know where the basis vectors end up after transformation, you know where any vector ends up as a linear combination of the basis vectors
- for the first (right-most) matrix in a multiplication, you can interpret each column as where a basis vector ends up
- then to multiply the second matrix, you need to do more work
  - the reason we could just assume the first column is the x basis vector in the first matrix is ultimately a consequence of the x value being 1 and the y value being 0
  - now though, we need to do the full multiplication
  - but it's still intuitive because now you just use both columns of the matrix
  - you end up with [x * col1, y * col2], which becomes the first column of the "composition" matrix of the two matrices
  - this is equivalent to the row-by-column multiplication algorithm
  - this also gives the intuition around why matrix multiplication is *not* commutative: some transformations like scaling + rotation can be done in either order, but rotation + shear is sensitive to the order that it's performed in 
