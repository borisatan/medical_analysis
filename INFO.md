

# What is a voxel?

A **Voxel** is a 3d cube and is the building block of the CT scan

It is the 3D version of a pixel. A CT scan is a 3D grid of voxels, and each voxel stores one number. Voxels aren't always perfect cubes: they can be box-shaped if the spacing differs between axes.

- **CT voxels:** the number is an intensity showing how dense the tissue is at that spot, in Hounsfield units. Air is around -1000 and bone is much higher.
- **Mask voxels:** the number is 0 or 1, meaning outside or inside the structure, such as a femur.

**In scan s1379:** the grid is 279 × 279 × 490 voxels. Each voxel is 1.5 mm wide on every axis, so here they are true cubes.

# What is "affine"?
The **affine** is a matrix that explains where each voxel sits in the real world. It converts array positions into real world coordinates

**Why it's needed:** the voxel array is only a 3D grid of numbers, with no information about how big each voxel is, where the grid sits or which way is left. The affine stores that information alongside the image.

**What it contains:** it's a 4×4 matrix, and two parts of it matter:

- **Top-left 3×3 block:** this holds the voxel size and the direction of each axis, including any rotation or flip.
- **Last column, top three values:** this is the offset, meaning the physical position of voxel (0, 0, 0).
- **Bottom row:** this is always 0, 0, 0, 1 and is only there to make the maths work.

To get a physical position, multiply the voxel index by the 3×3 block, then add the offset.

**In scan s1379:**

- **Voxel size:** the block has 1.5 on the diagonal and zeros elsewhere, so each voxel is 1.5 mm wide on every axis, with no rotation.
- **Starting point:** the offset is about (-216, -70, 26), which is where voxel (0, 0, 0) sits in physical space.
- **Worked example:** moving to voxel 278 on the first axis gives 278 × 1.5 = 417 mm. Adding that to -216 gives about 201 mm.
- **Orientation:** the R A S orientation also comes from the affine. It means that increasing the indices moves towards the patient's right, front (anterior) and top (superior).

**Example matrix:** this is the affine for scan s1379, with the offsets rounded to two decimals:

```
[  1.5    0      0     -216.09 ]
[  0      1.5    0      -70.09 ]
[  0      0      1.5     26.00 ]
[  0      0      0        1    ]
```

- **Voxel size and direction:** the 1.5s on the diagonal mean each voxel is 1.5 mm wide on each axis. They're positive, so increasing the indices moves towards the patient's right, front and top. The zeros around them mean there's no rotation.
- **Offset:** the last column shows that voxel (0, 0, 0) sits at (-216.09, -70.09, 26) mm.
- **Bottom row:** this is always 0, 0, 0, 1.

**Example: where is voxel (10, 20, 30)?**

| Axis | Index × voxel size | Plus offset | Physical position |
|---|---|---|---|
| x | 10 × 1.5 = 15 | 15 − 216.09 | -201.09 mm |
| y | 20 × 1.5 = 30 | 30 − 70.09 | -40.09 mm |
| z | 30 × 1.5 = 45 | 45 + 26 | 71 mm |

So voxel (10, 20, 30) is at (-201.09, -40.09, 71) mm.

**How a flip changes it:** if the top-left value were -1.5 instead of 1.5, increasing the first index would move towards the patient's left. The orientation would then read L A S instead of R A S. That's why comparing affines matters, as left and right can be swapped even when two images have the same shape.

**Why the name:** in maths, an affine transformation can scale, rotate, shear and shift, but it keeps straight lines straight and parallel lines parallel. That suits a scanner grid, which can be placed and tilted but not bent.

**Why it matters for this project:** a femur mask only lines up with the CT voxel for voxel if both have the same shape and the same affine. If they differ, the same index points to different places in the body, so a metal check could look in the wrong spot without raising any error.
