#  HW7：纹理表面重建

## Task 7.1 运行程序实现点云的清理

​		运行 examples/task7/task7_1_mesh_clean.cc 生成三维网格。命令行参数$ task7_1_mesh_clean –t10 –c100 /examples/data/surface_mesh.ply/examples/data/surface_mesh_clean.ply
​		其中-t10 表示删除最长边和最短遍大于 10 的三角形，-c1000 表示删除顶点个
数少于 1000 的独立区域。

 **运行结果：**

​		经过对比可以发现，清理之后对于一些突出的部分进行了清理。

![image-20220116222330215](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220116222330215.png)

![image-20220116222800924](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220116222800924.png)

![image-20220116222830483](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220116222830483.png)

## Task 7.2 运行程序实现纹理贴图

运行 examples/task7/task7_2_texturing.cc 生成纹理图像。命令行参数$ task7_2_texturing ./examples/data/sequence_scene::undistorted ./examples/data/sequence_scene/surface_mesh_clean.ply ./examples/data/sequence_scene

## Task 7.3 推导全局颜色公式

![task7.3](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/task7.3.jpg)

## Task 7.4 推导泊松表面重建线性方程

![image-20220122164215802](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220122164215802.png)