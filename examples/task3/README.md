# HW3：运动恢复结构（SFM）

[TOC]

## Task3.1调试程序

### 1、程序输出内容分析

#### 1.1输入无序图像

![image-20211127164844556](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127164844556.png)

#### 1.2图像连接图构建

##### 计算所有影响特征

![image-20211127165103087](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127165103087.png)

##### 影像两两匹配

![image-20211127165246926](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127165246926.png)

#### 1.3初始化

##### 选择一对相机并初始化相机内参

![image-20211127171234794](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127171234794.png)

##### Tracks滤波 & FULL BA

![image-20211127172200573](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127172200573.png)

#### 1.4重建新的视角

##### 单相机捆绑调整

![image-20211127172417796](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127172417796.png)

##### Tracks 重建及滤波

![image-20211127172626683](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127172626683.png)

##### 全局捆绑调整

![image-20211127172736764](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127172736764.png)

#### 1.5重建结果

![image-20211127172938799](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127172938799.png)

### 2、查看点云数据

```bash
# 在终端中输入以下命令安装CloudCompare
snap install core
sudo snap install cloudcompare
```

![image-20211127181256156](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211127181256156.png)

## Task3.2理解增量SFM的流程

### 1、增量BA伪代码

![](https://gitee.com/lpengsu/pic-go/raw/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20211128230451.jpg)

### 2、思考全局的SFM会不会有误差累积和漂移

​       **答**：会导致误差累积和漂移。每恢复一个pose会做一次局部BA，当恢复的pose数量占模型的一定比例时，会做全局BA。尽管有全局BA，但随着时间进行，误差仍然是在累积的。

### 3、计算增量SFM的复杂度，思考怎么加速

**答**：复杂度为O(n^4^)

​         加速主要从使用GPU进行加速、减少全局BA次数、改进BA算法等方面进行加速

![image-20211128222451074](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211128222451074.png)

![image-20211128222809534](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211128222809534.png)

### 4、对函数进行理解

1. add_exif_to_view --从图像交换文件中读取图像的焦距作为初始值；

2. sfm::bundler::Intrinsics::compute() --进行图像之间的两两匹配；

   代码中使用的应该是**bundler_features.cc 中的 compute函数**

3. sfm::bundler::Tracks::compute()--将所有的特征匹配对转化成 Tracks；

   代码中使用的应该是**bundler_tracks.cc 中的 compute函数**

4. sfm::bundler::InitialPair::compute()—选择初始的一对图像，进行增量的 BA；

   代码中使用的应该是**bundler_init_pair.cc 中的 compute_pair函数**

**类 sfm::bundler::Incremental**

1. triangulate_new_tracks() :当有新的相机姿态被重建之后，重建新的 track；
2. invalidate_large_error_tracks()：根据重投影误差去掉质量差的 track；
3. bundle_adjustment_full()：全局的 BA，同时优化所有的相机参数和三维点的坐标；
4. find_next_views()：找到新的相机视角用于重建，新的视角应该满足能够看到最多的地图点；
5. reconstruct_next_view()：重建新的相机的姿态(ransac_based_p3p)；
6. bundle_adjustment_single_cam(): 对单个视角进行 BA。
