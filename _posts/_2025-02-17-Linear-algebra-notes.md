Going through 3Blue1Brown's series as a refresher

**Video 4**

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

Also worth noting this is just 2D to 2D

**Video 5**

This video introduces 3D vectors and mentions the possibility of transformations between different dimenson vectors.
Jumping ahead a bit, but geometrically the intuition seems to be that if you have a 3x2 matrix, this can act as a function on 2D vectors 
that projects them into 3D space. But frankly my intuition is already breaking down wrt the component multiplication. 
Say you transform the x-basis vector. According to the intuition presented in the previous video, you should just take the first column. 
But that's not the case? Does that intuition only work for understanding NxN matrix multiplication?
Ah okay so I suppose in that case, the first column represents where the x-basis vector projects to and second column for the y-vector. You end up projecting both into 3D vectors. Then if you have (1,1) this is where (1,1) would be projected into 3D space by this matrix / is the linear combination of the transformation of the two original basis vectors.

I also find it interesting that the videos keep talking about rotations and whatnot, and how that's feasible with real numbers and yet
complex analysis kind of reworked my brain to think that only *i* can rotate. But it makes perfect sense right? if you move the [1,0] vector to [0,1] that's 90 degrees. I'm too dumb to grok the relationship between these vector rotations and imaginary scalar rotations (probably relevant that complex numbers are basically 2D numbers)

He mentioned the importance of 3D matrices in robotics because rotations can get complex but you can break them down into simpler ones. 
This is probably? relevant for some joint movements where maybe it's easier to rotate about one axis at a time. (Although cool to think about the kinematics in an obstacled environment where rotating one at a time fails but doing the rotations simultaneously or in a different order works). He mentions this breakdown but he hasn't talked yet about how to decompose(?) a matrix into "core" parts yet. 

He's also glossed over *which* matrices represent rotations vs scales vs shears. My guess is that rotations are basically just stuff like [[0,1], [1,0]] which will point the x vector at y and y vector at x - oh no! This is actually like a reflection and rotation. Rotation would be [[0,1], [-1,0]] and now *that's* funny because the fact that only one of them becomes negative really screams to me of *i* lurking somewhere in the math
