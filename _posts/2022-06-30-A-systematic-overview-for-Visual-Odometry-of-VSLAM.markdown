---
layout: post
read_time: true
show_date: true
title:  A systematic overview for Visual Odometry of VSLAM
date:   2022-06-30 19:13:20 UTC+8
description: A systematic overview for Visual Odometry of VSLAM.
img: posts/20220630/Learning_1000_points_per_iteration.jpg
tags: [machine learning, coding, neural networks]
author: Armando Maynez
github:  amaynez/Perceptron/
mathjax: yes
---
##  Visual Odometry技术 （Of VSLAM)



[toc]

### 什么是SLAM

​		SLAM是Simultaneous localization and mapping缩写，意为“同步定位与建图”[^1]。它是指搭载了特定传感器的主体，如机器人或者无人机等，在**没有**关于环境的**先验**知识的情况下，在运动的过程中建立环境的模型。SLAM的概念早在1986年[^2]就提出了。然而，早期的SLAM往往依赖价格昂贵或专门定制的传感器，例如激光雷达，声呐或立体相机，这项技术并未走入市场。随着算法和算力的不断发展，廉价的相机逐渐成为一种取代激光雷达等复杂设备进行SLAM的可能 。那么，如果涉及到的传感器主要为相机，那么就称为“视觉(Visual)SLAM”， 也就是标题所叙述的、这篇博客主要探讨的VSLAM。



### 经典视觉SLAM框架

如图1所示，下面是经典视觉SLAM框架的主要组成结构：一个SLAM系统主要由**Visual Odometry**(**视觉里程计**，**VO**)，**Optimization**(**后端优化**),**Loop Closing**(**回环检测**), **Mapping**(**建图**)这四个部分组成。 

![framework](framework.jpg)

<center> 图1 经典视觉SLAM框架 </center>

其中，VO能够通过相邻帧间的图像估计相机运动，并回复场景的空间结构。然而，仅仅有VO是不够的。VO其实就有点像马尔科夫链（Markov Chain）那样，只关注当前状态和未来状态的基本联系，不具有记忆性。这样一来，VO由于只有🐟的记忆，而每次的估计又会有一定偏差，每次估计的相机位姿运动偏差在机器人或者无人机运动过程中不断累加，形成累计偏移（**Accumulating Drift**）这些累计偏移有的时候会带来极为糟糕的后果。

如图2所示，设想一下，如果在估计的时候认为相机顺时针运动了90度，而实际上相机仅仅运动了89度，这样一来在一个空的矩形的房间里所作出的定位可能会因此不断远离相机的实际位置，而建立出来的地图也很可能会无法封闭。

![OIP-C](OIP-C.jpg)

<center> 图2 逐渐增大的偏移和无法闭合的地图 </center>

因此，我们需要在一个更宏观的视角下审视并且修正这些偏移。这样就引入了**后端优化**和**回环检测**这两个部分。

在**后端优化**中，则需要考虑相对更加长远的目标：在解决“如何从图像估计相机运动”的基础上从带有噪声的数据中估计整个系统的状态，以及这个状态估计的不确定性有多大，同时使得得到的相机位姿在全局上尽可能保持一致。相比于VO部分，Optimization往往没有那么可见，面对的只有数据，而不必关心数据的来自于激光雷达还是单目相机，又在视觉里程计之后，因此叫做后端。但是还是要注意的是，这里的前后端和web应用（如J2EE中的Servlet）中的前后端有所不同，要分清楚两者之间的区别。

在**回环检测**部分，最主要判断的还是机器人或者无人机是否达到之前到过的位置，如果检测到了闭环（往往是通过图像相似度判断实现），就会把信息提供给后端进行处理来得到一个全局一致的估计。用一个不太准确但是我自己觉得非常形象的比喻来说，这个过程就像是用把一个个用小棍子（预测的轨迹）穿起来的珠子（估计的点）头尾相接到一起保持中间各个珠子距离不变一样。现阶段应用最广的回环检测方法是词袋模型（Bag-of-Words），之后会详细介绍。

下面将对VSLAM框架中的VO技术进行具体介绍。

### Visual Odometry

如上所述，Visual Odometry主要是计算图像帧之间 的相机位姿关系，也即通过拍摄图像，估计出相机的运行位置和姿态信息。根据所使用相机的类型，我们可以把VO分为单目VO和立体VO[^3]，其中单目VO主要使用单目相机来获取环境的2D信息；而立体VO如RGB-D相机和双目相机在获取画面外能够直接通过结构光或者ToF获取场景深度信息或通过计算获得的场景深度信息（类似人眼）

在实际操作中，由于RGB-D相机由于很容易受到自然光的干扰，同时对于噪声的鲁棒性较差，本身价格也比较高不利于推广，因此主要用于室内SLAM；而双目相机的精度和深度方向上的量程受到基线长度，也即两个相机间距离的影响（但是做的宽一个是容易形变导致误差，一个是相机太宽影响运动），同时disparity map的计算要消耗大量的资源，往往需要GPU或者FPGA来加速，在深度上的测量很难达到令人满意的效果。

因此，单目相机SLAM技术便是这篇博客所要探讨的主要内容。在实践中，VO算法主要分为**特征点法**和**直接法**两类。

**特征点法**是通过汇总图像中所有有代表性的点的移动来预测相机的整体移动情况。由于通过矩阵在整个图像的层面来判断运动是十分困难的（LK光流需要强假设），因此我们可以用另一种图像的表现形式，也就是图像的特征来描述图片，减少不必要的信息（特征也可以看作是图像的主成分）。尽管特征点在面对墙体或者其他角点不显著（salient）的区域时可能难以识别[^4]，但是在绝大多数场景下都能够找到充足的特征点来对帧间运动做出一个大致的估计。

传统的寻找特征点的方法主要包括Harris角点（参考BUAASE_CV_hw_set2）、FAST角点[^5]等，这些经典的角点识别算法提出的时间较早，在图像变化幅度较大的情况下不够稳定。近年来不断发展的局部特征识别往往不仅匹配角点（或者也可以说兴趣点）本身，还会为角点提供相应的描述子（descriptor）来说明特征点的朝向和大小等信息。例如，SIFT就是一个十分经典的算法，能够对关照、尺度以及旋转都有很好的鲁棒性。然而，随着识别效果而来的还有巨大的计算量。与SfM不同，SLAM要求实时性，因此在课上熟知的SIFT很少被应用到SLAM的实际应用中。

那么有没有什么能够协调好准确率、鲁棒性以及计算量，使之能够适配SLAM的算法呢？当然有！这就是在SLAM中大名鼎鼎的ORB-SLAM，如图所示，就像YOLO一样，ORB也更新了很多版，证明了其强大的生命力。

![image-20220630133322628](C:/Users/hyj/AppData/Roaming/Typora/typora-user-images/image-20220630133322628.png)

<center> 图3 orb各个版本的论文（图源本人） </center>

ORB（Oriented FAST and Rotated BRIEF）,是目前最快速稳定的特征点检测和提取算法，许多图像拼接和目标追踪技术利用ORB特征进行实现[^6]。ORB-SLAM 是西班牙 Zaragoza 大学的 Raúl Mur-Arta 编写的视觉 SLAM 系统，目前开源在[raulmur/ORB_SLAM: A Versatile and Accurate Monocular SLAM (github.com)](https://github.com/raulmur/ORB_SLAM)上。正如GitHub上的md所述， ORB是一个通用高效的单目SLAM解决方案（后面的ORB2、ORB3支持了更多相机，但是这里先讨论单目的ORB）。

就像刚刚提到的，FAST角点以速度快而著称。已FAST-9为例，在像素点为中心的一个半径等于3像素的离散化的Bresenham圆找9个连续的像素点，如果它们们的像素值要么都比中心点加上一个阈值t大，要么都比中心点减去一个阈值t小，那么这个点就是一个角点。注意到FAST只用到了一个圆而没有具体的方向，事实上对于旋转缺乏鲁棒性；此外，由于它固定取半径为3的圆，因此存在尺度问题：可能一些在远处看是角点的位置放大后周围像素趋于一致而不再是角点了。ORB对FAST的改进或者拓展，主要是为其增加了其尺度不变性以及旋转不变性。

尺度不变性主要是通过图像金字塔例如**Gaussian pyramid**向下采样（使用高斯核对其进行卷积，然后对卷积后的图像进行下采样，反复迭代），是一种从粗糙到不断精细的过程。图像金字塔是单个图像的多尺度表示法，由一系列原始图像的不同分辨率版本组成。金字塔的每个级别都由上个级别的图像下采样版本组成。下采样是指图像分辨率被降低，比如图像按照 1/2 比例下采样。因此一开始的 4x4 正方形区域现在变成 2x2 正方形。图像的下采样包含更少的像素，并且以 1/2 的比例降低大小。这样一来，上面提到的角点放大丢失的问题就能够得到解决。

![Image result for Gaussian pyramid](https://tse1-mm.cn.bing.net/th/id/OIP-C.yUkPZoP-jdQ22-n9QmnSQgHaGo?w=199&h=180&c=7&r=0&o=5&dpr=1.14&pid=1.7)

<center> 图4 Gaussian pyramid 降采样 </center>

旋转不变性主要依靠ORB_SLAM的灰度质心法来处理：

灰度质心法首先要选择某个图像块B然后将图像块B的矩m定义为

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200329170932906.png)

那么可以找到图像块B的质心C:

![[公式]](https://www.zhihu.com/equation?tex=m_%7B00%7D+%3D+%5Csum_%7Bx%2C+y+%5Cin+B%7D+I+%28x%2C+y%29%2C+%5Cqquad+m_%7B10%7D+%3D+%5Csum_%7Bx%2C+y+%5Cin+B%7D+x+%2A+I+%28x%2C+y%29%2C+%5Cqquad+m_%7B01%7D+%3D+%5Csum_%7Bx%2C+y+%5Cin+B%7D+y+%2A+I+%28x%2C+y%29+%5C%5C+C+%3D+%5Cleft%28%5Cfrac%7Bm_%7B10%7D%7D%7Bm_%7B00%7D%7D%2C%5Cfrac%7Bm_%7B01%7D%7D%7Bm_%7B00%7D%7D%5Cright%29+%5Ctag%7B2%7D+)

方向向量可以通过将图像块的几何中心和它的质心连接在一起得到，所以可以定义特征点的方向为：

![[公式]](https://www.zhihu.com/equation?tex=%5Ctheta+%3D+arctan%28%5Cfrac%7Bm_%7B01%7D%7D%7Bm_%7B10%7D%7D%29+%5Ctag%7B3%7D+)

这样一来，FAST就有了尺度和方向的描述，就成为了Oriented FAST。

那么，有了关键点以后，我们就需要对每个点计算描述子。ORB中使用的描述子是Rotated BRIEF，是BRIEF的一种改进算法。BRIEF 是 Binary Robust Independent Elementary Features 的简称，它的作用是根据一组关键点创建二进制特征向量，又称为二进制特征描述符，是仅包含 1 和 0 的特征向量。在 BRIEF 中 每个关键点由一个二进制特征向量描述，该向量一般为 128-512 位的字符串，其中仅包含 1 和 0。由于向量中的每个值都是0或者1，因此BRIEF很适合用在像SLAM这样对实时性要求高同时资源又特别受限的技术上。BRIEF流程简单实时性较好，论文中生成512个描述子用时8.18ms[^6]，并且其描述子是二进制码，其匹配也比较快。但是，当BRIEF对于旋转过大时，比如超过30度时，匹配正确率迅速下降直到45度时为0。因此，和之前提到的FAST一样，BRIEF也不满足图像的尺度旋转不变性，因此，为使特征满足尺度不变性， Rotated BRIEF 算法同样**构建图像金字塔**。

值得一提的是论文中提到的steered BRIEF 来增加其旋转不变性[^6]：所谓steered BRIEF就是对挑选出的点对加上一个旋转角度θ。对于任何一个特征点来说，它的BRIEF描述子是一个长度为𝑛的二值码串，这个二值码串是由特征点邻域𝑛个点对生成的。

在代码实现的过程中我们可以使用OpenCV的库函数来辅助进行Oriented FAST的检测和BRIEF 描述子的计算。

```c++
//-- 第一步:检测 Oriented FAST 角点位置
  chrono::steady_clock::time_point t1 = chrono::steady_clock::now();
  detector->detect(img_1, keypoints_1);
  detector->detect(img_2, keypoints_2);

//-- 第二步:根据角点位置计算 BRIEF 描述子
descriptor->compute(img_1, keypoints_1, descriptors_1);
  descriptor->compute(img_2, keypoints_2, descriptors_2);
  chrono::steady_clock::time_point t2 = chrono::steady_clock::now();
  chrono::duration<double> time_used = chrono::duration_cast<chrono::duration<double>>(t2 - t1);
  cout << "extract ORB cost = " << time_used.count() << " seconds. " << endl;

  Mat outimg1;
  drawKeypoints(img_1, keypoints_1, outimg1, Scalar::all(-1), DrawMatchesFlags::DEFAULT);
  imshow("ORB features", outimg1);

//当然，也可以自己实现一个版本
void ComputeORB(const cv::Mat &img, vector<cv::KeyPoint> &keypoints, vector<DescType> &descriptors) {
  const int half_patch_size = 8;
  const int half_boundary = 16;
  int bad_points = 0;
  for (auto &kp: keypoints) {
    if (kp.pt.x < half_boundary || kp.pt.y < half_boundary ||
        kp.pt.x >= img.cols - half_boundary || kp.pt.y >= img.rows - half_boundary) {
      // outside
      bad_points++;
      descriptors.push_back({});
      continue;
    }

    float m01 = 0, m10 = 0;
    for (int dx = -half_patch_size; dx < half_patch_size; ++dx) {
      for (int dy = -half_patch_size; dy < half_patch_size; ++dy) {
        uchar pixel = img.at<uchar>(kp.pt.y + dy, kp.pt.x + dx);
        m10 += dx * pixel;
        m01 += dy * pixel;
      }
    }

    // angle should be arc tan(m01/m10);
    float m_sqrt = sqrt(m01 * m01 + m10 * m10) + 1e-18; // avoid divide by zero
    float sin_theta = m01 / m_sqrt;
    float cos_theta = m10 / m_sqrt;

    // compute the angle of this point
    DescType desc(8, 0);
    for (int i = 0; i < 8; i++) {
      uint32_t d = 0;
      for (int k = 0; k < 32; k++) {
        int idx_pq = i * 32 + k;
        cv::Point2f p(ORB_pattern[idx_pq * 4], ORB_pattern[idx_pq * 4 + 1]);
        cv::Point2f q(ORB_pattern[idx_pq * 4 + 2], ORB_pattern[idx_pq * 4 + 3]);

        // rotate with theta
        cv::Point2f pp = cv::Point2f(cos_theta * p.x - sin_theta * p.y, sin_theta * p.x + cos_theta * p.y)
                         + kp.pt;
        cv::Point2f qq = cv::Point2f(cos_theta * q.x - sin_theta * q.y, sin_theta * q.x + cos_theta * q.y)
                         + kp.pt;
        if (img.at<uchar>(pp.y, pp.x) < img.at<uchar>(qq.y, qq.x)) {
          d |= 1 << k;
        }
      }
      desc[i] = d;
    }
    descriptors.push_back(desc);
  }

  cout << "bad/total: " << bad_points << "/" << keypoints.size() << endl;
}
```



在取得了所有匹配好的点对后，就可以通过点对之间的关系来估计单目相机的运动，这部分由于涉及到大量的对极几何约束而且相关的代码都可以在OpenCV中找到，比如**cvFindFundamentalMat**和**findEssentialMat**等函数可以直接免去大量的理解，主要还是理清基本的三角测量原理和本质矩阵的应用，这里就不再过多展开了。

![See the source image](https://ts1.cn.mm.bing.net/th/id/R-C.878c7f07c2baa8549cca6c58597d098e?rik=sUOIUk5zbJXukA&pid=ImgRaw&r=0)

<center> 图5 三角测量原理和对极约束 </center>



上述便是特征点法操作的主要流程。特征点法很清晰直接，但是还是有一些问题。例如，即使ORB速度已经相当快了，也仍然需要20ms的时长，如果想要做到一个30帧的实时SLAM，那么就需要每一帧在平均33ms左右。这样一来大部分的时间开销都会花在特征点的提取上。

此外，使用特征点后，图像本身就被丢弃了。尽管特征点能够在某种意义上反映图像的情况，但是一张图像毕竟有十几万像素，而特征点往往只有几百个，在一些情况下可能难以反映可能有用的图像信息。

同时，在一些特征点不是太显著的场合，比如沿着墙体的方向，或者是空无一物的走廊，单靠特征点很难识别出相机的运动。这些都有可能会带来相关的问题。

因此，在一些情况下，直接法可能更加使用，其中一个主要的代表就是LSD-SLAM。相比于特征点法，直接法并不要求一一对应的匹配，只要先前的点在当前图像当中具有合理的投影残差，就认为这次投影是成功的：成功与否主要取决于对地图点深度以及相机位姿的判断，并不在于图像局部看起来是什么样子。直接法节省特征提取与匹配的大量时间，易于移植到嵌入式系统中，以及与IMU进行融合，其中LK光流技术就是一个著名的近似例子。



### Lucas–Kanade光流

Lucas–Kanade光流算法是一种两帧差分的光流估计算法。它由Bruce D. Lucas 和 Takeo Kanade提出[^7]。

**Optical flow**, 或者说光流，是一种运动模式，这种运动模式指的是一个物体、表面、边缘在一个视角下由一个观察者（比如眼睛、摄像头等）和背景之间形成的明显移动。它计算两帧在时间t 到t + δt之间每个每个像素点位置的移动。 由于它是基于图像信号的泰勒级数，这种方法称为差分，这就是对于空间和时间坐标使用偏导数。 图像约束方程可以写为
$$
I (x ,y ,z,t ) = I (x + δx ,y + δy ,z + δz ,t + δt )
$$
其中，I(x, y,z, t) 为在（x,y,z）位置的体素。 我们假设移动足够的小，那么对图像约束方程使用泰勒公式，我们可以得到：

![img](https://pic4.zhimg.com/80/v2-aa8060a6a519f61f51b9a1eede99314b_720w.jpg)

忽略高阶无穷小项，可以得到：

![img](https://pic3.zhimg.com/80/v2-800e00be3b3cfca9de37e5e1e3cc11d6_720w.jpg)

利用滑动窗口可以得到一个超定方程，



![image-20220630200833343](C:/Users/hyj/AppData/Roaming/Typora/typora-user-images/image-20220630200833343.png)

使用最小二乘法求解就可以得到：

![image-20220630200913212](C:/Users/hyj/AppData/Roaming/Typora/typora-user-images/image-20220630200913212.png)

这也是进行估计的基础。此外，LK光流算法也可以通过前面提到的金字塔方法来进行优化，这样一来能够避免运动速度过快、图像整体结构发生变化等问题。

在实现中，我们也可以使用高斯牛顿法来计算光流。核心函数如下（cpp）

```cpp
void OpticalFlowTracker::calculateOpticalFlow(const Range &range) {
    // parameters
    int half_patch_size = 4;
    int iterations = 10;
    for (size_t i = range.start; i < range.end; i++) {
        auto kp = kp1[i];
        double dx = 0, dy = 0; // dx,dy need to be estimated
        if (has_initial) {
            dx = kp2[i].pt.x - kp.pt.x;
            dy = kp2[i].pt.y - kp.pt.y;
        }

        double cost = 0, lastCost = 0;
        bool succ = true; // indicate if this point succeeded

        // Gauss-Newton iterations
        Eigen::Matrix2d H = Eigen::Matrix2d::Zero();    // hessian
        Eigen::Vector2d b = Eigen::Vector2d::Zero();    // bias
        Eigen::Vector2d J;  // jacobian
        for (int iter = 0; iter < iterations; iter++) {
            if (inverse == false) {
                H = Eigen::Matrix2d::Zero();
                b = Eigen::Vector2d::Zero();
            } else {
                // only reset b
                b = Eigen::Vector2d::Zero();
            }

            cost = 0;

            // compute cost and jacobian
            for (int x = -half_patch_size; x < half_patch_size; x++)
                for (int y = -half_patch_size; y < half_patch_size; y++) {
                    double error = GetPixelValue(img1, kp.pt.x + x, kp.pt.y + y) -
                                   GetPixelValue(img2, kp.pt.x + x + dx, kp.pt.y + y + dy);;  // Jacobian
                    if (inverse == false) {
                        J = -1.0 * Eigen::Vector2d(
                            0.5 * (GetPixelValue(img2, kp.pt.x + dx + x + 1, kp.pt.y + dy + y) -
                                   GetPixelValue(img2, kp.pt.x + dx + x - 1, kp.pt.y + dy + y)),
                            0.5 * (GetPixelValue(img2, kp.pt.x + dx + x, kp.pt.y + dy + y + 1) -
                                   GetPixelValue(img2, kp.pt.x + dx + x, kp.pt.y + dy + y - 1))
                        );
                    } else if (iter == 0) {
                        // in inverse mode, J keeps same for all iterations
                        // NOTE this J does not change when dx, dy is updated, so we can store it and only compute error
                        J = -1.0 * Eigen::Vector2d(
                            0.5 * (GetPixelValue(img1, kp.pt.x + x + 1, kp.pt.y + y) -
                                   GetPixelValue(img1, kp.pt.x + x - 1, kp.pt.y + y)),
                            0.5 * (GetPixelValue(img1, kp.pt.x + x, kp.pt.y + y + 1) -
                                   GetPixelValue(img1, kp.pt.x + x, kp.pt.y + y - 1))
                        );
                    }
                    // compute H, b and set cost;
                    b += -error * J;
                    cost += error * error;
                    if (inverse == false || iter == 0) {
                        // also update H
                        H += J * J.transpose();
                    }
                }

            // compute update
            Eigen::Vector2d update = H.ldlt().solve(b);

            if (std::isnan(update[0])) {
                // sometimes occurred when we have a black or white patch and H is irreversible
                cout << "update is nan" << endl;
                succ = false;
                break;
            }

            if (iter > 0 && cost > lastCost) {
                break;
            }

            // update dx, dy
            dx += update[0];
            dy += update[1];
            lastCost = cost;
            succ = true;

            if (update.norm() < 1e-2) {
                // converge
                break;
            }
        }

        success[i] = succ;

        // set kp2
        kp2[i].pt = kp.pt + Point2f(dx, dy);
    }
}

```



需要注意的是，LK光流技术有一个较强的假设：要求图像的亮度恒定，就是同一点随着时间的变化，其亮度不会发生改变。因此，相比于SIFT等特征提取算法而言，LK的速度较快，但损失了一定的精度和鲁棒性，在具体使用的过程中还是要见仁见智。



要想对SLAM有深入的研究，处理cv相关的代码能力外，也少不了扎实的数学基础。例如，四元数、李群李代数（尤其是SO(3)和SE(3)这些特殊的群)。同时，还需要有一定的图形学基础和非线性优化的能力。在不断探索的过程中，后端优化的Bundle Adjustment与Loop Closure都亟需丰富的统计学知识和实操上手的优化能力，不管是传统的KF、EKF还是流行的NLO，限于篇幅这篇博客不能一一揽括，因此就集中先把前端技术的思考记录下来，综合成一篇文字，以飨读者。







### Acknowledgements and References

[^1]: Liu H ,  Zhang G ,  Bao H . A survey of monocular simultaneous localization and mapping[J]. Journal of Computer-Aided Design & Computer Graphics, 2016.
[^2]: Smith, Randall & Cheeseman, Peter. (1987). On the Representation and Estimation of Spatial Uncertainty. The International Journal of Robotics Research. 5. 10.1177/027836498600500404.
[^3]: G. Yang, Y. Wang, J. Zhi, W. Liu, Y. Shao and P. Peng, "A Review of Visual Odometry in SLAM Techniques," *2020 International Conference on Artificial Intelligence and Electromechanical Automation (AIEA)*, 2020, pp. 332-336, doi: 10.1109/AIEA51086.2020.00075.
[^4]: B. X. Hon, H. Tian, F. Wang, B. M. Chen and T. H. Lee, "A customized fastslam algorithm using scanning laser range finder in structured indoor environments," 2013 10th IEEE International Conference on Control and Automation (ICCA), 2013, pp. 640-645, doi: 10.1109/ICCA.2013.6565202.
[^5]: Trajkovic, Miroslav & Hedley, Mark. (1998). Fast Corner Detection. Image and Vision Computing. 16. 75-87. 10.1016/S0262-8856(97)00056-5.
[^6]: R. Mur-Artal, J. M. M. Montiel and J. D. Tardós, "ORB-SLAM: A Versatile and Accurate Monocular SLAM System," in IEEE Transactions on Robotics, vol. 31, no. 5, pp. 1147-1163, Oct. 2015, doi: 10.1109/TRO.2015.2463671.
[^7]: Baker S , Matthews I . Lucas-Kanade 20 Years On: A Unifying Framework[J]. International Journal of Computer Vision, 2004, 56(3):221-255.
