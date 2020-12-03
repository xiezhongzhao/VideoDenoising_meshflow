[TOC]

## **视频降噪性能测试**

1. 测试平台

   * Inter(R) Core(TM) i-7700 CPU @ 3.6GHz 
   * 内存（RAM）: 16.0 GB
   * 系统：Windows 10 64位操作系统

2. 测试结果

   业务要求是在骁龙600系列的手机上，达到1080P 30FPS。

视频帧5帧：

| 视频分辨率 | KLT (ms) | Mesh Motion (ms) | Absolute Motion (ms) | Target Frame (ms) | Total (ms) |
| :--------: | :------: | :--------------: | :------------------: | :---------------: | ---------- |
|    720     |    60    |        10        |          10          |        50         | 130        |
|    1080    |    70    |        15        |          20          |        115        | 220        |

视频帧3帧：

| 视频分辨率 | KLT (ms) | Mesh Motion (ms) | Absolute Motion (ms) | Target Frame (ms) | Total (ms) |
| :--------: | :------: | :--------------: | :------------------: | :---------------: | ---------- |
|    720     |    60    |        10        |          4           |        31         | 105        |
|    1080    |    70    |        15        |          7           |        73         | 165        |

视频帧2帧：

| 视频分辨率 | KLT (ms) | Mesh Motion (ms) | Absolute Motion (ms) | Target Frame (ms) | Total (ms) |
| :--------: | :------: | :--------------: | :------------------: | :---------------: | :--------: |
|    720     |    60    |        10        |          2           |        23         |     95     |
|    1080    |    70    |        15        |          3           |        52         |    140     |

------



## **视频降噪技术步骤分析**

1. 特征点的光流跟踪 （KLT）

   对视频图像中的特征点进行快速检测（Fast corner detection），再对特征点进行KLT光流追踪（Good features to track），但是这种稀疏特征追踪方法会由于特征点分布不均匀，导致没有检测出特征的位置跟踪失败。

2. Mesh网格特征点均匀分布（Mesh Motion）

   MeshFlow将捕获的特征点均匀分布在视频帧上，具体是将图像分成若干网格，用每个网格的顶点运动矢量代替顶点附近的特征运动分量，对于附近没有特征点运动的顶点，运用全局顶点的平均运动分量代替。(具体内容参见文献[1])。

3. 像素的绝对运动 （Absolute Motion）

   网格顶点的像素点随着时间不断积累固定位置的运动矢量，这种方式可以很好的近似该像素点真实的运动轨迹。（具体内容参见文献[2]）。

4. 帧间融合（Target Frame）

   帧间融合是对同一空间位置的像素进行平均去噪，但是由于有些像素不能平均，如果这些像素没有对齐进行融合会导致严重的重影，一般会根据实验情况来选取一个合适的阈值。（具体内容参见文献[3]）。

------



## 视频降噪改进思路

视频降噪的主要性能瓶颈在于运动估计和帧间融合这两块，运动估计需要改进光流跟追算法，在保证效果的同时，降低算法复杂度；帧间融合的耗时取决于融合的帧数，帧数越多去噪效果越好，但是耗时越长，因此需要做权衡。帧间融合的优化完全可以通过工程方式降低耗时。

1. 运动估计的优化思路（Google Pixel 2）

   在KLT金字塔稀疏光流追踪过程中，在更粗的金字塔层的整数精度的运动矢量足够精确来初始化运动矢量搜索到下一个更细的层，因此，除了最后一层外，可以去除所有不必要的插值来计算亚像素对齐误差。最后，为了增加运动搜索范围，减少迭代次数，使用最粗层的水平和垂直的一维投影，通过互相关来估计全局运动。这种优化方法时间比原来缩短4倍。（具体内容参见文献[4]）

2. 帧间融合的优化思路 （工程优化指令集、多线程、OpenCL、OpenGL等）

   帧间融合可以通过工程优化大幅度降低耗时。

------



## **参考文献**

1. MeshFlow: Minimum Latency Online Video Stabilization
2. SteadyFlow: Spatially Smooth Optical Flow for Video Stabilization
3. MESHFLOW VIDEO DENOISING
4. Real-Time Video Denoising On Mobile Phones

