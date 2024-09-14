# Camera-Calibration

This is the fourth assignment of my computer vision course in the university. The assignment comes with partially completed Python programs and the tasks are to estimate a planar projective transformation for each of the planes of a calibration grid, generate 2D-3D correspondences for corners on a calibration grid, estimate a projection matrix for a camera, decompose a projection matrix into a camera calibration matrix and a matrix composed of the rigid body motion, and estimate an essential matrix for a pair of cameras.


### To calibrate the camera, I have done the following features: 

1. estimating a planar projective transformation for each of the planes of a calibration grid,
    In this stage, the function calibrate2D() is implemented. The steps are as follows:
      1. form the matrix equation Ap = b for the X-Z plan
          I set up empty array for both b and A corresponding to the XZ-plane. There are 7 unknowns to be solved and so I set Axz to be with 7 coloumns.
          With a forloop looping from 4 times, I stacked up a 8x7 matrix Axz and a vector bxz with 8 entires.
      2. solve for the planar projective transformation using linear least squares
          I used np.linalg.lstsq(Axz,bxz,rcond=None)[0] to solve for the planar projective transformation using linear least squares,
          the function returned a vector, and I insert a value 1 at the end.
          After that, I reshape the vector into 3x3 matrix Hxz.
      3. form the matrix equation Ap = b for the Y-Z plane
          I set up empty array for both b and A corresponding to the YZ-plane. There are 7 unknowns to be solved and so I set Ayz to be with 7 coloumns.
          With a forloop looping from 4 times, I stacked up a 8x7 matrix Ayz and a vector byz with 8 entires.
      4. solve for the planar projective transformation using linear least squares
          I used np.linalg.lstsq(Ayz,byz,rcond=None)[0] to solve for the planar projective transformation using linear least squares,
          the function returned a vector, and I insert a value 1 at the end.
          After that, I reshape the vector into 3x3 matrix Hyz.

2. generating 2D-3D correspondences for corners on a calibration grid,
    In this stage, the function gen_correspondences() is implemented. The steps are as follows:
      1. define 3D coordinates of all the corners on the 2 calibration planes, I manually pick 80 3D points on the XZ plane in world coordinates and do the same for the YZ plane,
         by using a two for-loops from the coordinates. Since the points are actually separated by a value of 1, I increased it every time by 1.
         Finally, I combined the ref3D_xz and ref3D_yz into ref3D.
      2. project corners on the calibration plane 1 onto the image: I first created  an empty array with shape (0,2).
          I get the XZ coordinates from ref3D by deleting the second column, and then insert a column with value 1 at the end of the matrix XZ
          Finally, I set up a for loop to get the image coordinates one-by-one through multiplying the  matrix Hxz with the XZ points,
          afterwards stacking up into ref_xz and this give the corners on the calibration plane (XZ plane) onto the image.
      3. project corners on the calibration plane 2 onto the image
          This is similar to the implementation of "project corners on the calibration plane 2 onto the image".
          I get the YZ coordinates from ref3D by deleting the first column, and then insert a column with value 1 at the end of the matrix YZ
          Finally, I set up a for loop to get the image coordinates one-by-one through multiplying the  matrix Hxz with the YZ points,
          afterwards stacking up into ref_yz and this give the corners on the calibration plane (YZ plane) onto the image.
      4. locate the nearest detected corners
          I first combined the ref_xz and ref_yz to form ref2D. Then I used find_nearest_corner(ref2D, corners) to find the closet points of corners and return the value to ref2D.


3. estimating a projection matrix for a camera,
    In this stage, the function calibrate3D() is implemented. The steps are as follows:
      1. form the matrix equation Ap = b for the camera, I set up the vector b from the 160 points of ref2D and matrix A from the 160 points of ref3D and red2D
          by using a for-loop to stacking up
      2. solve for the projection matrix using linear least squares by calling the function np.linalg.lstsq(A,b,rcond=None)[0]
          it returns a vector P so I inserting 1 at the end of the vector P. Fianlly, it return the 3x4 matrix P by reshaping it.

4. decomposing a projection matrix into a camera calibration matrix and a matrix composed of the rigid body motion, and
    In this stage, the function decompose_P() is implemented. The steps are as follows:
      1. extract the  3 x 3 submatrix from the first 3 columns of P through np.delete()
      2. perform QR decomposition on the inverse of [P0 P1 P2] through np.linalg.inv() on the extracted matrix and use the function np.linalg.qr() to perform QR decomposition
      3. obtain K as the inverse of R through np.linalg.inv()
      4. obtain R as the transpose of Q by Q.T
      5. for the purpose of normalizing K, K is divided by the scaler k_22
          then two more checkings are done:
            i. If the k_00 is negative, then the first row of K and the first column of Ro is multiplied by -1
            ii. If the k_11 is negative, then the second row of K and the second column of Ro is multiplied by -1
      6. obtain T from P3, first get the p3 from the fourth column of the RT, then multiplied the inverse of K with p3 and divdie it by a scalar alpha as we have T = (1/alpha) * K^(-1)*\vec(p_3)
      Finally, reshape T as 3x1 and insert it into the last column of Ro and return RT

5. estimating an essential matrix for a pair of cameras.
    In this stage, the function compose_E() is implemented. The steps are as follows:
      1. compute the relative rotation R:
          I first extract the R1 and R2 from RT1 and RT2 respectively by deleting the last column of the RT1 and RT2.
          Then I find the rotation matrix by multiplying R2 and the transpose of R1 since we have R = (R2)(R1)^T
      2. compute the relative translation T
          Then I extract the T1 and T2 from RT1 and RT2 respectively by accessing the last column of the RT1 and RT2.
          Since the relative translation T = T2 - RT1, I did the operation through T = np.subtract(T2, R @ T1)
      3. compose E from R and T:
          For the ease of calculating E = [T]x R, I used an np array to create the [T]x and finally get the desired result through Tx.dot(R)
