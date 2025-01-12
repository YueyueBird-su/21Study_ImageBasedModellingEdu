





# HW1:相机模型与对极几何

## Task1.1 基本矩阵操作

1. 引入的头文件

   ```c++
   #include <math/matrix_svd.h>
   #include "math/matrix.h"
   #include "math/vector.h"
   ```

2. 主要功能

   - 矩阵、向量的初始化

   - 矩阵元素的访问（单个值、某一行、某一列）

   - 奇异值分解

     *函数定义：*A = U S V^t^

     ```c++
     matrix_svd (Matrix<T, M, N> const& mat_a, Matrix<T, M, N>* mat_u,
         Matrix<T, N, N>* mat_s, Matrix<T, N, N>* mat_v, T const& epsilon)
     ```

     ![image-20211112223738160](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211112223738160.png)

## Task1.2 针孔相机模型

### Vec2d projection( )函数

![image-20211112224430337](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211112224430337.png)

***编程结果***见附录

### Vec3d pos_in_world( )函数

![image-20211112230034899](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211112230034899.png)

### Vec3d dir_in_world( )函数

​	旋转矩阵中的第三行即为相机在世界坐标系中的朝向

![image-20211112230156668](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211112230156668.png)

## Task1.3 直接线性变化法

```c++
 [直接线性变换法]
 双目视觉中相机之间存在对极约束
                       p2'Fp1=0,
 其中p1, p2 为来自两个视角的匹配对的归一化坐标，并表示成齐次坐标形式，
 即p1=[x1, y1, z1]', p2=[x2, y2, z2],将p1, p2的表达形式带入到
 上式中，可以得到如下表达形式
          [x2] [f11, f12, f13] [x1, y1, z1]
          [y2] [f21, f22, f23]                = 0
          [z2] [f31, f32, f33]
 进一步可以得到
 x1*x2*f11 + x2*y1*f12 + x2*f13 + x1*y2*f21 + y1*y2*f22 + y2*f23 + x1*f31 + y1*f32 + f33=0
 写成向量形式
               [x1*x2, x2*y1,x2, x1*y2, y1*y2, y2, x1, y1, 1]*f = 0,
 其中f=[f11, f12, f13, f21, f22, f23, f31, f32, f33]'
 由于F无法确定尺度(up to scale, 回想一下三维重建是无法确定场景真实尺度的)，因此F秩为8，
 这意味着至少需要8对匹配对才能求的f的解。当刚好有8对点时，称为8点法。当匹配对大于8时需要用最小二乘法进行求解
   [x11*x12, x12*y11,x12, x11*y12, y11*y12, y12, x11, y11, 1]
   [x21*x22, x22*y21,x22, x21*y22, y21*y22, y22, x21, y21, 1]
   [x31*x32, x32*y31,x32, x31*y32, y31*y32, y32, x31, y31, 1]
 A=[x41*x42, x42*y41,x42, x41*y42, y41*y42, y42, x41, y41, 1]
   [x51*x52, x52*y51,x52, x51*y52, y51*y52, y52, x51, y51, 1]
   [x61*x62, x62*y61,x62, x61*y62, y61*y62, y62, x61, y61, 1]
   [x71*x72, x72*y71,x72, x71*y72, y71*y72, y72, x71, y71, 1]
   [x81*x82, x82*y81,x82, x81*y22, y81*y82, y82, x81, y81, 1]
现在任务变成了求解线性方程
               Af = 0
（该方程与min||Af||, subject to ||f||=1 等价)
通常的解法是对A进行SVD分解，取最小奇异值对应的奇异向量作为f分解
本项目中对矩阵A的svd分解并获取其最小奇异值对应的奇异向量的代码为
   math::Matrix<double, 9, 9> V;
   math::matrix_svd<double, 8, 9>(A, nullptr, nullptr, &V);
   math::Vector<double, 9> f = V.col(8);
[奇异性约束]
  基础矩阵F的一个重要的性质是F是奇异的，秩为2，因此有一个奇异值为0。通过上述直接线性法求得
  矩阵不具有奇异性约束。常用的方法是将求得得矩阵投影到满足奇异约束得空间中。
  具体地，对F进行奇异值分解
               F = USV'
  其中S是对角矩阵，S=diag[sigma1, sigma2, sigma3]
  将sigma3设置为0，并重构F
                       [sigma1, 0,     ,0]
                 F = U [  0   , sigma2 ,0] V'
                       [  0   , 0      ,0]
```

![image-20211112231438925](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211112231438925.png)

![image-20211112231412332](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211112231412332.png)

***编程结果***见附录

## Task1.4 RANSAN

### calc_ransac_iterations( )函数

![image-20211113093747637](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211113093747637.png)

### find_inliers( )函数

用于查找满足阈值的内点

![image-20211113094120550](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211113094120550.png)

***编程结果***见附录

## Task1.5 求解相机参数

### 求解本征矩阵E

![image-20211113094732304](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211113094732304.png)

![image-20211113104323141](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211113104323141.png)

![image-20211113104621403](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211113104621403.png)

**三角化**：求解世界坐标

![image-20211113104517226](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211113104517226.png)

***编程结果***见附录

## Task1.6 特征匹配函数及算法

### Maching 类

- oneway_match( ) 单向匹配
- twoway_match( ) 双向匹配
- remove_inconsistent_matches( ) 删除双向匹配的不一致匹配
- count_consistent_matches( ) 计算有效匹配的数量
- combine_results( ) 整合匹配结果

### Exhaustive_matching 类

- init( )
- pairwise_match( ) 匹配所有特征类型，产生单个匹配结果
- pairwise_match_lowers( ) 匹配 N 个最低分辨率的特征并返回匹配的数量

### Cascade_hashing 类

利用**哈希表**对之前实现的函数进行加速（没咋看懂）

### Lowe-ratio 算法

> 原文链接：https://blog.csdn.net/u010368556/article/details/82939404

​		为了进一步筛选匹配点，来获取优秀的匹配点，这就是所谓的“去粗取精”。一般会采用Lowe’s算法来进一步获取优秀匹配点。
​		为了排除因为图像遮挡和背景混乱而产生的无匹配关系的关键点，SIFT的作者Lowe提出了**比较最近邻距离与次近邻距离的SIFT匹配方式**：取一幅图像中的一个SIFT关键点，并找出其与另一幅图像中欧式距离最近的前两个关键点，在这两个关键点中，如果最近的距离除以次近的距离得到的比率ratio少于某个阈值T，则接受这一对匹配点。因为对于错误匹配，由于特征空间的高维性，相似的距离可能有大量其他的错误匹配，从而它的ratio值比较高。显然降低这个比例阈值T，SIFT匹配点数目会减少，但更加稳定，反之亦然。
​		Lowe推荐ratio的阈值为0.8，但作者对大量任意存在尺度、旋转和亮度变化的两幅图片进行匹配，结果表明ratio取值在0. 4~0. 6 之间最佳，小于0. 4的很少有匹配点，大于0. 6的则存在大量错误匹配点，所以建议ratio的取值原则如下:

ratio=0. 4：对于准确度要求高的匹配；

ratio=0. 6：对于匹配点数目要求比较多的匹配；

ratio=0. 5：一般情况下。

```c++
if (static_cast<float>(nn_result.dist_1st_best)
    / static_cast<float>(nn_result.dist_2nd_best)
    > MATH_POW2(options.lowe_ratio_threshold))
    continue;
```

![Without lowe-ratio](https://gitee.com/lpengsu/pic-go/raw/master/img/matching_featureset2.png)

*图1：未使用Lowel-ratio*

![Lowe-ratio](https://gitee.com/lpengsu/pic-go/raw/master/img/matching_featureset.png)

*图2：使用Lowel-ratio* ratio=0.8

![](https://gitee.com/lpengsu/pic-go/raw/master/img/matching_featureset3.png)

*图3：使用Lowel-ratio* ratio=0.6

![](https://gitee.com/lpengsu/pic-go/raw/master/img/matching_featureset4.png)

*图4：使用Lowel-ratio* ratio=0.4

## Task1.7 SIFT 特征检测模块

1. 构建尺度空间
2. 构建高斯差分尺度空间
3. 高斯差分尺度空间极值点精确定位
4. 去除噪声点和边缘点
5. 关键点主方向寻找
6. 主方向对齐
7. 特征直方图计算
8. 特征归一化

![](https://gitee.com/lpengsu/pic-go/raw/master/img/example.sift.jpg)

# 附录：

task1.2

- [x] projection coord:
   0.208188 -0.035398
  result should be:
   0.208188 -0.035398
- [x] cam position in world is:
   -0.0948544 -0.935689 0.0943652
  result should be: 
   -0.0948544 -0.935689 0.0943652
- [x] cam direction in world is:
   -0.0155846 0.00757181 0.999854
  result should be: 
   -0.0155846 0.00757181 0.999854

------

task1.3

- [x] Fundamental matrix after singularity constraint is:
   -0.0315082 -0.63238 0.16121
  0.653176 -0.0405703 0.21148
  -0.248026 -0.194965 -0.0234573
- [x] Result should be: 
  -0.0315082 -0.63238 0.16121
  0.653176 -0.0405703 0.21148
  -0.248026 -0.194965 -0.0234573

------

task1.4

- [x] inlier number: 272
  F
  : -0.00961384 -0.0309071 0.703297
  0.0448265 -0.00158655 -0.0555796
  -0.703477 0.0648517 -0.0117791
- [x] result should be: 
  inliner number: 272
  F: 
  -0.00961384 -0.0309071 0.703297
  0.0448265 -0.00158655 -0.0555796
  -0.703477 0.0648517 -0.0117791

------

task1.5

- [x] EssentialMatrix result is -0.00490744 -0.0146139 0.34281
  0.0212215 -0.000748851 -0.0271105
  -0.342111 0.0315182 -0.00552454
- [x] EssentialMatrix should be: 
  -0.00490744 -0.0146139 0.34281
  0.0212215 -0.000748851 -0.0271105
  -0.342111 0.0315182 -0.00552454
- [x] Result of 4 candidate camera poses shoule be 
  R0:
  -0.985336 0.170469 -0.0072771
  0.168777 0.980039 0.105061
  0.0250416 0.102292 -0.994439
  t0:
   0.0796625 0.99498 0.0605768
  R1: 
  -0.985336 0.170469 -0.0072771
  0.168777 0.980039 0.105061
  0.0250416 0.102292 -0.994439
  t1:
  -0.0796625 -0.99498 -0.0605768
  R2: 
  0.999827 -0.0119578 0.0142419
  0.0122145 0.999762 -0.0180719
  -0.0140224 0.0182427 0.999735
  t2:
  0.0796625 0.99498 0.0605768
  R3: 
  0.999827 -0.0119578 0.0142419
  0.0122145 0.999762 -0.0180719
  -0.0140224 0.0182427 0.999735
  t3: 
  -0.0796625 -0.99498 -0.0605768
- [x] P1: 0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P1 for fist pose should be
  0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P2: -0.957966 0.165734 -0.00707496 0.0774496
  0.164089 0.952816 0.102143 0.967341
  0.0250416 0.102292 -0.994439 0.0605768
- [x] P2 for fist pose should be
   -0.957966 0.165734 -0.00707496 0.0774496
  0.164089 0.952816 0.102143 0.967341
  0.0250416 0.102292 -0.994439 0.0605768

- [x] A: -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  0.963181 -0.14443 -0.200031 -0.0648336
  -0.164975 -0.956437 -0.0669352 -0.969486

- [x] A for first pose should be:
  -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  0.963181 -0.14443 -0.200031 -0.0648336
  -0.164975 -0.956437 -0.0669352 -0.969486
- [x] X:3.20431 -2.77102 17.1956
  X for first pose should be:
  3.2043116948585566 -2.7710180887818652 17.195578538234088
- [x] P1: 0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P1 for fist pose should be
  0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P2: -0.957966 0.165734 -0.00707496 -0.0774496
  0.164089 0.952816 0.102143 -0.967341
  0.0250416 0.102292 -0.994439 -0.0605768
- [x] P2 for fist pose should be
   -0.957966 0.165734 -0.00707496 0.0774496
  0.164089 0.952816 0.102143 0.967341
  0.0250416 0.102292 -0.994439 0.0605768

- [x] A: -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  0.963181 -0.14443 -0.200031 0.0648336
  -0.164975 -0.956437 -0.0669352 0.969486

- [x] A for first pose should be:
  -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  0.963181 -0.14443 -0.200031 -0.0648336
  -0.164975 -0.956437 -0.0669352 -0.969486
- [x] X:-3.20431 2.77102 -17.1956
  X for first pose should be:
  3.2043116948585566 -2.7710180887818652 17.195578538234088
- [x] P1: 0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P1 for fist pose should be
  0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P2: 0.972054 -0.0116257 0.0138463 0.0774496
  0.0118752 0.971991 -0.0175699 0.967341
  -0.0140224 0.0182427 0.999735 0.0605768
- [x] P2 for fist pose should be
   -0.957966 0.165734 -0.00707496 0.0774496
  0.164089 0.952816 0.102143 0.967341
  0.0250416 0.102292 -0.994439 0.0605768

- [x] A: -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  -0.974974 0.015425 0.194363 -0.0648336
  -0.0113787 -0.972637 -0.0178253 -0.969486

- [x] A for first pose should be:
  -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  0.963181 -0.14443 -0.200031 -0.0648336
  -0.164975 -0.956437 -0.0669352 -0.969486
- [x] X:1.32001 -1.14151 7.08367
  X for first pose should be:
  3.2043116948585566 -2.7710180887818652 17.195578538234088
- [x] P1: 0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P1 for fist pose should be
  0.972222 0 0 0
  0 0.972222 0 0
  0 0 1 0
- [x] P2: 0.972054 -0.0116257 0.0138463 -0.0774496
  0.0118752 0.971991 -0.0175699 -0.967341
  -0.0140224 0.0182427 0.999735 -0.0605768
- [x] P2 for fist pose should be
   -0.957966 0.165734 -0.00707496 0.0774496
  0.164089 0.952816 0.102143 0.967341
  0.0250416 0.102292 -0.994439 0.0605768

- [x] A: -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  -0.974974 0.015425 0.194363 0.0648336
  -0.0113787 -0.972637 -0.0178253 0.969486

- [x] A for first pose should be:
  -0.972222 0 0.180123 0
  -0 -0.972222 -0.156584 -0
  0.963181 -0.14443 -0.200031 -0.0648336
  -0.164975 -0.956437 -0.0669352 -0.969486
- [x] X:-1.32001 1.14151 -7.08367
  X for first pose should be:
  3.2043116948585566 -2.7710180887818652 17.195578538234088
- [x] Correct pose found!
- [x] R: 0.999827 -0.0119578 0.0142419
  0.0122145 0.999762 -0.0180719
  -0.0140224 0.0182427 0.999735
- [x] t: 0.0796625 0.99498 0.0605768
- [x] Result should be: 
  R: 
  0.999827 -0.0119578 0.0142419
  0.0122145 0.999762 -0.0180719
  -0.0140224 0.0182427 0.999735
  t: 
  0.0796625 0.99498 0.0605768

