# HW6: 表面重建

## Task6.1 实现点云到网格的重建

### 1、运行稠密点云生成深度图

​		通过执行task4的代码产生各视角的深度图。

### 2、进行深度图融合生成点云

​		将深度图通过4-2融合成点云，并保存到指定文件夹，通过软件查看点云数据如下：

![image-20220104205347358](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220104205347358.png)

### 3、执行表面重建命令生成surface mesh

​		通过表面重建命令生成surface mesh 以及 通过代码，处理不满足要求的面片，可以发现一些不合理的面片被清理掉了。 

![image-20220104205540604](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220104205540604.png)

![image-20220104205606245](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220104205606245.png)

## Task6.2 实现八叉树的广度优先遍历

​		octree.cc 中定义的类 Iterator 用于实现对八叉树节点的遍历。类成员 next_node()实现了深度优先遍历。参考深度优先遍历在类函数 next_bread_first()中实现广度优先搜索。

​		广度优先遍历是指从根节点开始，从左到右输出每一层的节点。则会有以下思路：从根节点出发，遍历第一个子节点之后，遍历第二个分支的节点，直到遍历完所有分支。再遍历到下一个子节点，再遍历所有分支。直到所有节点遍历结束。

```c++
Octree::Iterator::next_bread_first(void)
{
    /*todo implement bread first search here*/

    // 如果下一个叶子节点为空，则返回第一个节点的子节点
    if(this->next_leaf() == nullptr){
        return this->first_leaf()->children;
    }

    this->current = this->next_leaf();
    this->level = this->level - 1;
    this->path = this->path >> 3;

    return this->current;
}
```

## Task6.3 遍历八叉树的所有体素

1. examples/task6/task6-1_surface_reconstruction.cc: line 86 处(TODO 提示)遍历所有的 voxels, 并且打印出每个 voxel 的符号距离值。

   ​		**查看octree的源码，可以发现其中实现了一个get_voxels的函数，能够获取与vector类似的自定义结构。使用以下代码可遍历所有的Voxel值。**

   ```c++
   fssr::IsoOctree::VoxelVector voxels = octree.get_voxels();
   
   std::cout << octree.get_voxels().size() << std::endl;
   
   for(int i = 0; i < voxels.size(); i++){
   	if(i % 12 ==0){
   		std::cout << std::endl;
   	}
           std::cout << voxels[i].second.value << "\t";
   }
   ```

   ![image-20220105163449611](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220105163449611.png)

2. 试着改变 voxel 的符号距离值（同时增加一定量或者减少一定量），然后对比重建的结果，尝试解释一下这些操作对重建结果的影响。

   ​		**尝试设置三种组合 【- 10 ， 0， 10， 100】**

   - *不进行修改*

   - *增加10*

     ![image-20220105163847742](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220105163847742.png)

   - *减少10*

     ![image-20220105164232654](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220105164232654.png)

   - 增加100

     ![image-20220105165235357](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220105165235357.png)

   - *总结*

     通过输出的ply文件可以发现，表面重建出来的mesh图像没有区别，即不管同时增加还是减少，都不会改变对mesh的重建效果。

     ![image-20220105165412259](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220105165412259.png)

## Task6.4 证明Delaunay Triangulation的最大化最小角特性

> 参考：https://zhuanlan.zhihu.com/p/375542978

​		该特性是指：在散点集可能形成的三角剖分中，Delaunay三角剖分所形成的三角形的最小角最大。

即：

![image-20220105170703476](https://gitee.com/lpengsu/pic-go/raw/master/originalHost:%20'gitee.com',/image-20220105170703476.png)