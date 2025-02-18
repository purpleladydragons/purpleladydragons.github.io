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
complex analysis kind of reworked my brain to think that only _i_ can rotate. But it makes perfect sense right? if you move the [1,0] vector to [0,1] that's 90 degrees. I'm too dumb to grok the relationship between these vector rotations and imaginary scalar rotations (probably relevant that complex numbers are basically 2D numbers)

He mentioned the importance of 3D matrices in robotics because rotations can get complex but you can break them down into simpler ones. 
This is probably? relevant for some joint movements where maybe it's easier to rotate about one axis at a time. (Although cool to think about the kinematics in an obstacled environment where rotating one at a time fails but doing the rotations simultaneously or in a different order works). He mentions this breakdown but he hasn't talked yet about how to decompose(?) a matrix into "core" parts yet. 

He's also glossed over *which* matrices represent rotations vs scales vs shears. My guess is that rotations are basically just stuff like [[0,1], [1,0]] which will point the x vector at y and y vector at x - oh no! This is actually like a reflection and rotation. Rotation would be [[0,1], [-1,0]] and now *that's* funny because the fact that only one of them becomes negative really screams to me of _i_ lurking somewhere in the math

**Video 6**

Determinant

Oh wow right from the get go he is making the determinant make sense. It represents how area changes after transformation. Crazy. Never remotely understood how determinants were relevant but wow. 

Holy shit lol. Okay so I vaguely remember normalizing matrices and I wonder if the determinant has to end up being 1 in that case? 

Ah here he goes. Negative values in the determinant. Basically it's flipping the orientation of space / reflecting.

Oh shoot lol I forgot what the determinant even was. I thought it was just the diagonal multiplication but that's not it. It's all that wacky addition stuff too. But I guess that still makes sense.

His intuition around negative determinants is great. Scaling down towards 0 and then punching through to larger magnitude negative numbers.

Ah he's working through the actual formula now. For diagonal matrix, it's straightforward: just the new rectangular area. If even one of the non-diagonal values are zero, then you end up with a parallelogram of same area as the rectangle. So that term is still 0 etc. If both values are non-zero, then you subtract their product. It's hard for me to _really_ intuit why as they get bigger the determinant / area should get smaller. But I guess it's because they "stretch" it along the diagonal which actually shrinks the area. Because notably in the diagram below, you can see that they increase the side lengths of the parallelogram, but in a way that is dominated by the non-relevant area you have to subtract. Idk fully but sure 

<img width="486" alt="image" src="https://github.com/user-attachments/assets/0c29a683-b44c-4411-bd52-57602def244b" />

He finishes with a question: why is det(M1*M2) = det(M1) * det(M2)? My intuition is that it's because it measure how much the matrix scales up space and scaling is transitive. 

** Video 7**

Talking about inverse matrices. It's already clocking why some aren't invertible. If you destroy information (zeroing out a variable), then you can't get it back.

There's no inverse if determinant is 0. There's the case where any of the diagonal values are 0 which makes sense because that's destroying info. But what about when it shakes out that ad = bc? I guess that's like shearing it into a line? It's possible there's still a solution to the inverse problem if v happens to sit on this new squished space. 

Is it the case that non-invertible _iff_ det(A) = 0?

Ah cool this is related to rank apparently. If you squish 3D vector to 2D or 1D, those are different. Rank 2 means plane, rank 1 means line. 

The set of all possible outputs of your matrix is the column space. The span of the columns is the column space. This makes senes. If you have orthogonal vectors, great you can represent more space with them. But if not, then you can only add them onto the same line/plane. 

Now he's talking about null space / kernel. This is the set of vectors that map to the origin after transformation. If you have a map from 3D to 2D plane, then the set of vectors orthogonal to the plane will all map to the origin. Presumably relevant when you're solving equations of the form Ax = [0,0] which seems relevant for solving systems of linear equations

** Video 9 **

Dot product. Why does order not matter? He shows that with a symmetrical example, it obviously doesn't matter. But when you double one of the vectors, 
you can see that the other vector's projection doesn't really change and then you just double the length. Or you halve the length of the doubled vector to get same original result and then double it and same thing? Basically the angle doesn't change at all, just the magnitude along that angle so it doesn't matter which way you rotate / project.

So why does the numerical computation relate to the geometric projection interpretation?

Losing it a little bit but basically taking the dot product of... two vectors is like projecting the one vector onto the other because you can treat it as if you were projecting onto the unit length version of the one vector and then scaling along its line/dimension. And this works out because dot product is commutative and the projection of one vector onto another is equivalent to the opposite. Idk if that actually makes sense... 


