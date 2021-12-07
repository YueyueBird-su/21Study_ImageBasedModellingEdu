# HW4：稠密重建

## Task 4.1 推导相机内参矩阵的逆举证

**答**：根据题目所给条件，像素宽度和高度相同，根据公式
$$
\frac{f}{d} = \frac{x}{w}=\frac{y}{h}\\
其中f是相机的焦距，以像素为单位；\\d为物体到相机的距离，单位为米；\\x是物体在图像中的宽度，w为物体的实际宽度；\\y是物体在图像中的高度，而h是物体的实际高度。
$$
当像素的宽度x和高度y相等时，fx = fy = f；且由于相机光心点对应图像的中心，所以像片没有偏移，即x0 = y0；则推出相机内参矩阵为：
$$
K = 
\begin{bmatrix}
	f_x & 0 & 0\\
	0 & f_y & 0\\
	0 & 0 & 1\\
\end{bmatrix}
$$
则其逆矩阵为：
$$
K^{-1} = 
\begin{bmatrix}
	\frac{1}{f} & 0 & 0\\
	0 & \frac{1}{f} & 0\\
	0 & 0 & 1\\
\end{bmatrix}
$$

## Task 4.2 图像分辨率估计

**答**：根据Task 4.1 中的公式
$$
已知：\frac{f}{d} = \frac{x}{w}=\frac{y}{h} 则：\\
    	\frac{1}{x}=\frac{d}{f * w}\\
    	\frac{1}{y}=\frac{d}{f * h}\\
    	令：a = max(x,y)，当世界中物体为球时，有w = h = r，即：\\
    	\frac{1}{a} = \frac{d}{f * r}\\
    	其中 r 为世界中物体的半径\\
    	r = \frac{d}{f}*a
$$
其中，r为世界中物体的半径，d为物体到像中心点的距离，f为焦距，a为物体所占的像素宽度。

当固定物体大小仅占一个像素时，随着r增大，表示一个像素所表示的内容增多，即分辨率提高；而当r减小，一个像素所表示的内容减少，即分辨率降低。

## Task 4.3 推导求导公式

![微信图片_20211207153552](https://gitee.com/lpengsu/pic-go/raw/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20211207153552.jpg)

## Task 4.4 单个视角稠密重建结果

### 1、global_view_selection

- **GlobalViewSelection()**

  传入所有的**视角（Views）**、SFM生成的**特征**、**参数**设置（参考帧、候选帧个数…）对全局视角选取进行初始化构造。

- **performVS()**

  通过调用benefitFromView() 函数计算最好的多个视角

- **benefitFromView()**

  对两视角的Score进行计算，分别从两视角视差、分辨率大小、与其他已选视角间的视差三方面进行计算。

### 2、local_view_selection

局部视角选择是从全局视角中选取一些视角(最多4个)，局部视角是针对每个patch的。因此更加灵活，也就是每个patch的局部视角可以是不同的，但是全局视角都是相同的。

通过计算NCC值以及计算极平面之间的法向量，充分考虑分辨率差异以及视线之间的夹角以及与其他已选择的视角进行夹角计算，得出视角的分数，选出最大分数的几幅影像。

### 3、单幅图像重建流程

![微信图片_20211207164806](https://gitee.com/lpengsu/pic-go/raw/master/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20211207164806.jpg)

### 4、单幅图像重建结果

![image-20211207165014347](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211207165014347.png)

## Task 4.5 多幅图像重建结果

![image-20211207165046152](https://gitee.com/lpengsu/pic-go/raw/master/img/image-20211207165046152.png)