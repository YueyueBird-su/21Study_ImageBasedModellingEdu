# HW2: 帧间运动估计

## Task2.1 实现线性三角化算法

```c++
/* 实现线性三角化方法(Linear triangulation methods), 给定匹配点
 * 以及相机投影矩阵(至少2对），计算对应的三维点坐标。给定相机内外参矩阵时，
 * 图像上每个点实际上对应三维中一条射线，理想情况下，利用两条射线相交便可以
 * 得到三维点的坐标。但是实际中，由于计算或者检测误差，无法保证两条射线的
 * 相交性，因此需要建立新的数学模型（如最小二乘）进行求解。
 *
 * 考虑两个视角的情况，假设空间中的三维点P的齐次坐标为X=[x,y,z,1]',对应地在
 * 两个视角的投影点分别为p1和p2，它们的图像坐标为
 *          x1=[x1, y1, 1]', x2=[x2, y2, 1]'.
 *
 * 两幅图像对应的相机投影矩阵为P1, P2 (P1,P2维度是3x4),理想情况下
 *             x1=P1X, x2=P2X
 *
 * 考虑第一个等式，在其两侧分别叉乘x1,可以得到
 *             x1 x (P1X) = 0
 *
 * 将P1X表示成[P11X, P21X, P31X]',其中P11，P21，P31分别是投影矩阵P1的第
 * 1～3行，我们可以得到
 *
 *          x1(P13X) - P11X     = 0
 *          y1(P13X) - P12X     = 0
 *          x1(P12X) - y1(P11X) = 0
 * 其中第三个方程可以由前两个通过线性变换得到，因此我们只考虑全两个方程。每一个
 * 视角可以提供两个约束，联合第二个视角的约束，我们可以得到
 *
 *                   AX = 0,
 * 其中
 *           [x1P13 - P11]
 *       A = [y1P13 - P12]
 *           [x2P23 - P21]
 *           [y2P23 - P22]
 *
 * 当视角个数多于2个的时候，可以采用最小二乘的方式进行求解，理论上，在不存在外点的
 * 情况下，视角越多估计的三维点坐标越准确。当存在外点(错误的匹配点）时，则通常采用
 * RANSAC的鲁棒估计方法进行求解。
 */
```

## Task2.2 推导实现P3P

**P3p**: 从3对3D-2D的对应点中确定相机的朝向和位置

- 通常会产生4对解，需要用第4对匹配关系确定

- 一般的求解思路是首先计算点在相机中的3D坐标，然后通过ICP的方式计算相机姿态

- Kneip是一种close-form的P3p求解方式，主要思想是引入相机和世界坐标系的中间坐标系，计算它们之间的相对姿态和位置来得到相机的姿态，Kneip的优势是求解稳定、速度快

### 1、创建新的相机坐标系

​	注意tx、ty、tz的计算顺序

![image-20211121192932736](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121192932736.png)

### 2、创建新的世界坐标系

![image-20211121193115762](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121193115762.png)

### 3、相机中心和姿态在新建世界坐标系的表达

​		相机坐标系的tz 垂直于CP1P2这个平面，根据相似关系，求出新建相机坐标系在平面中的表达。

![image-20211121193307946](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121193307946.png)

### 4、将P3从新建世界坐标系转换到相机坐标系

![image-20211121194127064](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121194127064.png)

### 5、构建四次方程

![image-20211121194236904](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121194236904.png)

### 6、基于RANSAN的P3P算法

![image-20211121194424240](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121194424240.png)

## Task2.3 Levenberg-Marquardt

![image-20211121210237157](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121210237157.png)

![image-20211121210746134](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211121210746134.png)

![微信图片_20211123232017](https://gitee.com/lpengsu/pic-go/raw/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20211123232017.jpg)

![](https://gitee.com/lpengsu/pic-go/raw/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20211125115409.jpg)

## Task2.4 推导并实现雅可比矩阵

![](https://gitee.com/lpengsu/pic-go/raw/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20211125164607.jpg)

![image-20211125165306174](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211125165306174.png)

![image-20211125165329494](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211125165329494.png)

![image-20211125165355713](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211125165355713.png)

![image-20211125165423745](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211125165423745.png)

![image-20211125165447553](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211125165447553.png)

![image-20211125165512424](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211125165512424.png)

## Task2.5 双视SFM

分别经历了：

- 特征点提取与匹配
- 相机基础矩阵求解
- 相机姿态恢复
- 三维点三角测量
- 相机姿态与三维点坐标非线性优化

# 附录：编程结果

## 附录1

- [x]  trianglede point is :2.14598 -0.250569 6.92321
- [ ]  the result should be 2.14598 -0.250569 6.92321

## 附录2

- [ ] inverse k matrix: 1.02857 0 0

  0 1.02857 0

  0 0 1

- [x] solution 0: 
  0.255196 -0.870439 -0.420964 3.11336
  0.205378 0.474244 -0.856103 5.85437
  0.944825 0.132017 0.299795 0.427484

- [x] solution 1: 
  0.255196 -0.870439 -0.420964 3.11336
  0.205378 0.474244 -0.856103 5.85437
  0.944825 0.132017 0.299795 0.427484

- [x] solution 2: 
  0.999829 -0.00839409 -0.0164546 -0.0489098
  0.00840211 0.999965 0.000418264 -0.905048
  0.0164505 -0.000556447 0.999865 -0.0303771

- [x] solution 3: 
  0.975996 0.122883 0.17981 -1.42073
  -0.213276 0.706478 0.674835 -5.68456
  -0.0441065 -0.696985 0.715728 1.71503

  the result should be 
  solution 0:
  0.255193 -0.870436 -0.420972 3.11342
  0.205372 0.474257 -0.856097 5.85432
  0.944825 0.132022 0.299794 0.427496

  solution 1:
  0.255203 -0.870431 -0.420976 3.11345
  0.205372 0.474257 -0.856097 5.85432
  0.944825 0.132022 0.299794 0.427496

  solution 2:
  0.999829 -0.00839209 -0.0164611 -0.0488599
  0.00840016 0.999965 0.000421432 -0.905071
  0.016457 -0.000559636 0.999864 -0.0303736

  solution 3:
  0.975996 0.122885 0.179806 -1.4207
  -0.213274 0.706483 0.67483 -5.68453
  -0.0441038 -0.69698 0.715733 1.71501

- [ ] reproj err of solution 0 0.307975

- [ ] reproj err of solution 1 0.307975

- [x] reproj err of solution 2 3.23739e-07      【误差最小的就是正确的解算结果】

- [ ] reproj err of solution 3 0.00146693

------

2D-3D correspondences inliers: 99
Estimated pose: 
0.999539 0.029919 -0.00522629 -0.478246
-0.0297788 0.999241 0.0251186 -4.81399
0.00597385 -0.0249514 0.999671 0.0831591

The result pose should be:
0.99896 0.0341342 -0.0302263 -0.292601
-0.0339703 0.999405 0.0059176 -4.6632
0.0304104 -0.00488465 0.999526 -0.0283862

## 附录3

- [x] BA: #0  success, MSE 6.78947e-07 -> 5.44019e-08, CG  23, TRR 1000, MSE Ratio: 0.919873
  BA: #1  success, MSE 5.44019e-08 -> 5.17351e-08, CG  34, TRR 10000, MSE Ratio: 0.0490201
  BA: #2  success, MSE 5.17351e-08 -> 5.13398e-08, CG  43, TRR 100000, MSE Ratio: 0.00764002
  BA: #3  success, MSE 5.13398e-08 -> 5.11923e-08, CG  45, TRR 1e+06, MSE Ratio: 0.0028746
  BA: #4  success, MSE 5.11923e-08 -> 5.11783e-08, CG  41, TRR 1e+07, MSE Ratio: 0.000271943
  BA: #5  success, MSE 5.11783e-08 -> 5.11783e-08, CG  35, TRR 1e+08, MSE Ratio: 1.02694e-06
  BA: #6  success, MSE 5.11783e-08 -> 5.11783e-08, CG   9, TRR 1e+09, MSE Ratio: 2.04743e-08
  BA: #7  success, MSE 5.11783e-08 -> 5.11783e-08, CG   9, TRR 1e+10, MSE Ratio: 3.38104e-10

- [x] BA: Satisfied delta mse ratio threshold of 1e-08

- [x] Params after BA: 
    f: 0.979286
    distortion: -0.0647893, 0.100872
    R: 1 -0.000290652 -0.000651535
  0.000288844 0.999994 -0.00429066
  0.000652274 0.00429056 0.999994

    t: 0.000577118 0.0298317 0.0233335

- [x] Params after BA: 
    f: 0.97825
    distortion: -0.0705359, 0.117627
    R: 0.999808 -0.0125244 0.0150901
  0.0127235 0.999837 -0.0130887
  -0.0149232 0.0132782 0.999804

    t: 0.0803042 0.966335 0.0538348

- [x] points 3d: 
  1.32748, -1.14973, 7.14914
  0.0187157, 0.929084, 7.54932
  -1.13185, -1.24751, 7.26178
  -1.0112, -0.572129, 7.3779
  0.062655, -0.914864, 7.46404

## 附录4

​	**Result is :**

- [x] cam_x_ptr: 0.19312 0.0123983 0.000847141 0.129284 0.000847456 -0.0253633 0.0255783 0.944413 0.161936 

- [x] cam_y_ptr: -0.16782 -0.010774 -0.000736159 0.000847456 0.129523 0.0220405 -0.938992 -0.0240256 0.177293 

- [x] point_x_ptr: 0.12925 0.000495992 -0.0255451

- [x] point_y_ptr: 0.000963221 0.129745 0.02069

  **Result should be :**

- [x] cam_x_ptr: 0.195942 0.0123983 0.000847141 0.131188 0.000847456 -0.0257388 0.0260453 0.95832 0.164303

- [x] cam_y_ptr: -0.170272 -0.010774 -0.000736159 0.000847456 0.131426 0.0223669 -0.952795 -0.0244697 0.179883

- [x] point_x_ptr: 0.131153 0.000490796 -0.0259232

- [x] point_y_ptr: 0.000964926 0.131652 0.0209965

## 附录5

- [x] Loading ./examples/data/sequence/IMG_0191.JPG...
  Loading ./examples/data/sequence/IMG_0192.JPG...
  Focal length: 0.972222 0
  Focal length: 0.972222 0
  focal length: f1 0.972222 f2: 0.972222
  SIFT: Creating 4 octaves (0 to 4)...
  SIFT: Generating keypoint descriptors...
  SIFT: Generated 492 descriptors from 445 keypoints, took 2458ms.
  SIFT: Creating 4 octaves (0 to 4)...
  SIFT: Generating keypoint descriptors...
  SIFT: Generated 463 descriptors from 448 keypoints, took 2429ms.
  Image 1 (702x468) 492 descriptors.
  Image 2 (702x468) 463 descriptors.
  Consistent Sift Matches: 266

- [x] RANSAC-F: Running for 1000 iterations, threshold 0.0015...
  RANSAC-F: Iteration 1, inliers 29 (10.9023%)
  RANSAC-F: Iteration 4, inliers 230 (86.4662%)
  RANSAC-F: Iteration 8, inliers 261 (98.1203%)
  RANSAC-F: Iteration 23, inliers 262 (98.4962%)
  RANSAC-F: Iteration 324, inliers 263 (98.8722%)

- [x] F: 0.0016963 -0.0705411 -0.698898
  0.0543137 0.00206532 0.126149
  0.685457 -0.133537 0.00329399

  Number of Matching pairs is 263

- [x] E: -0.00499314 -0.0172686 0.342859
  0.0237567 -0.000765821 -0.0267099
  -0.342119 0.0311955 -0.00556201

- [x] P1: 0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0

  P2: -0.95832 0.163708 -0.0061403 0.0764011
  0.161803 0.951541 0.116643 0.966964
  0.0263835 0.117209 -0.992757 0.0679174

  A: -0.972222 0 0.180118 0
  -0 -0.972222 -0.156568 -0
  0.963815 -0.139297 -0.200623 -0.0622558
  -0.162736 -0.955687 -0.081525 -0.969366

  P1: 0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0

  P2: -0.95832 0.163708 -0.0061403 -0.0764011
  0.161803 0.951541 0.116643 -0.966964
  0.0263835 0.117209 -0.992757 -0.0679174

  A: -0.972222 0 0.180118 0
  -0 -0.972222 -0.156568 -0
  0.963815 -0.139297 -0.200623 0.0622558
  -0.162736 -0.955687 -0.081525 0.969366

- [x] Successful triangulation:  263 points

- [x] BA: #0  success, MSE 6.79034e-07 -> 5.68719e-08, CG  17, TRR 1000, MSE Ratio: 0.916246
  BA: #1  success, MSE 5.68719e-08 -> 5.65466e-08, CG  18, TRR 3000, MSE Ratio: 0.00571968

  BA: Satisfied delta mse ratio threshold of 1e-08
  BA: MSE 6.79034e-07 -> 5.61871e-08, 10 LM iters, 152 CG iters, 90ms.

- [x] Cam 0

- [x] Params before BA: 
    f: 0.972222
    distortion: 0, 0
    R: 1 0 0
  0 1 0
  0 0 1

    t: 0 0 0

- [x] Params after BA: 
    f: 0.972222
    distortion: 0, 0
    R: 0.999999 8.0384e-05 0.00142364
  -8.13786e-05 1 0.000903849
  -0.00142358 -0.000903945 0.999999

    t: -0.0158865 -0.00395483 0.00993855

- [x] Cam 1

- [x] Params before BA: 
    f: 0.972222
    distortion: 0, 0
    R: 0.999824 -0.0120624 0.0143949
  0.0123169 0.999767 -0.0177222
  -0.0141778 0.0178964 0.999739

    t: 0.0785839 0.994591 0.0679174

- [x] Params after BA: 
    f: 0.972222
    distortion: 0, 0
    R: 0.999845 -0.0120275 0.0128879
  0.0122652 0.999754 -0.0185016
  -0.0126622 0.0186568 0.999746

    t: 0.0937537 1.00071 0.0536728

- [x] points 3d: 
  ( 1.3239, -1.1441, 7.10012 )-->( 1.32034, -1.14422, 7.08088 )
  ( 0.01922, 0.913034, 7.42332 )-->( 0.019796, 0.914142, 7.44268 )
  ( -1.12621, -1.24291, 7.21142 )-->( -1.12065, -1.24188, 7.17981 )
  ( -1.00236, -0.569467, 7.2902 )-->( -0.999725, -0.57169, 7.2759 )
  ( 0.06407, -0.909536, 7.37132 )-->( 0.0642627, -0.91051, 7.35075 )



