# Basic Motion Planning And Overview

> 本质上是一个搜索问题，即对一个给定的函数，寻找最优解。相对于无人车而言，规划问题就是给定现在的状态，找到无人车移动的最优解。通常最优解目标函数F(x)定义。

规划问题设计三个领域，不同领域对问题的理解不同：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815162606438.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

> 从最简单问题出发，把运动规划抽象成一个路径查找问题，寻找最优值

* 通过启发式方式对搜索问题进行优化。breadth first search（BFS）和 depth first search（DFS）都属于non-information search，问题就在于不知道目标在哪里。

* A-star算法是大概知道红点在右边，定义一个启发式函数，该函数猜测距离目标还有多远，通过这种方法先搜索一些比较近的点，然后从这个点出发逐渐扩大搜索圈。
* A-star花费时间比广度优先算法时间更短，因为它有信息支持

现在的一些路径搜索算法本质上都是从A-star算法出发，需要知道目标函数的样子。目前，A-star算法还不能直接用在规划模块上，因为A-star算法本身要求对整个环境全知。而自动驾驶对周围环境是部分观察的。对于部分观察我们可以使用贪心算法，其实就是一个增量搜索，就是在看见的情况下尽量走好。

利用A-star算法对部分观察的数据进行控制规划。它利用当前能够看到的信息进行增量规划，A-star的特点是处理在看到的有限范围的条件下，如何到达预定地点的搜索问题方法。这种增量搜索很难通过一步步的迭代达到全局最优解。

在现实生活中，人类开车是很少做90度直角转弯的，这样的折线并没有考虑无人车运动过程中的运动模型和动力学模型。更进一步，可以通过平滑性曲线的方式来优化折线，换成一些较为平滑的曲线来完成

**上述介绍的搜索技术离自动驾驶运动规划还差多少？**

1. 在部分观察空间的动态障碍物，规划模块怎么处理动态障碍物是关键并且是有难度的。
2. 自动驾驶汽车能否按照规划的行驶，可能需要一个动态模型。
3. 缺少了遵守交通规则，它是道路安全的基本保证，将交通规则融入规划也是一个难点。
4. 实时计算，目前来说百度要求规划模块运转周期是100毫秒。

运动规划问题就是让自动驾驶车辆能够安全平稳到达终点，本质是一个三维规划问题，即 XY 坐标加上时间维度，叫做 3D Trajectory Optimization Problem（轨迹优化问题）。

从车辆动力学模型来说，维度需要进一步上升，因为涉及到车头的方向，车的转向角、加速度等问题。

无人车硬件系统除了汽车之外，还涉及很多传感器，传感器感知汽车周围环境，即使是这样也只是部分搜索环境。还有 GPS 接收器可以做定位，以及 IMU 惯性导航系统。

**运动规划可以获得两部分信息**，

1. 一部分是动态信息，包括从认知获得的信息，就是从感知模块和定位模块获得信息，

2. 一部分是静态信息，就是高精地图。

无人驾驶系统软件包括定位、感知、预测、运动规划和控制等。

车辆状态、交通灯信息、障碍以及障碍轨迹、导航、高精地图都是规划模块能获得的信息。

规划就是在这样的部分可见信息中给无人车找到一条轨迹。它不仅是一条路径，而是随着时间推移路径该怎么走，它包含两方面，① 路径信息，② 速度配置文件，需要保证速度和路径变化都是平滑的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815170124998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# Motion Planning with Autonomous Driving

> 决策规划问题都是从质点模型触发考虑，质点模型将运动轨迹当成一个点，这个点和无人车是不一样的。
>
> 假设把一个无人车看成一个点，这个点和另一个点不相撞，在数学定义上是点和点没有交集，但是在实际生活中一个车和一个车可是会相撞。

## Configuration Space

> 能够控制什么变量。对于刚体而言，不仅是XY坐标，还要有heading信息才能研究跟障碍物之间的关系。对于无人车来说有更多的变量。

**复杂性体现：**

1. Space Dimensionality
2. Geometric Complexity

我们知道规划问题中涉及一些约束条件：

1. Local Constraint
2. Differential Constraint
3. Global Constraint

> 运动规划是在连续空间的一种优化

对于连续空间过程的优化往往比较难，通常先将连续空间问题离散化表示，然后寻找对应的解决方案。– Roadmap，使用简单的连通图表示配置空间。

其他方法：

1. 除了Roadmap之外，还有Cell decomposition（网格分解方法）和Potential field（势场法）等路径规划方法。Cell decomposition将整个空间分割成一个个cell，通过cell的连接图表示自由空间的连接属性。Potential field就是直接用微分方法处理。
2. 一种常用的抽象连续空间的方法叫做PRM。它在整个配置空间随机采样一些点，如果点在障碍物上则去掉，然后将这些点连接起来。从点s到g的最短路径就可以利用A-Star算法进行求解。但是该方法要求是对全局感知，而无人车是一个部分感知的应用场景，因此有RRT的改进方法。

**改进：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815171035817.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

**RRT：**

通过这种方式离散化的线是不适合无人车行驶的，因为这些线的curvature不连续，甚至curvature都没有。针对这一问题MIT提出使用平滑曲线进行连接的方法。但是该方法得到的路径可能还是不够平滑，另外对动态障碍物的处理也存在问题。

**Lattice网络：**

* 传统
  * 在XY世界坐标系中，以1米为单位进行网格划分，然后用无人车可以行进的、曲率连续的曲线将起始点和目标点连接起来。
* 常用
  * 在sl坐标系下进行离散的方法Lattice in Frenet Frame

**Polynomial：**

> 使用路径-速度迭代优化的方法对Lattice方法进行优化

将问题降维，分成了path 和 speed两个维度逐渐优化，这是一种iterative的处理方式。

**Functional Optimization：**

对运动规划进行处理，对整个问题建模，设计相应的代价函数。二次规划就是其中一种常用的方法。

# Motion Planning with Environment

> 运动规划根据环境的变化在算法和处理方法上有很大的不同，涉及到模型建立、平滑优化和坐标转换以及障碍物投影等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815171639809.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

以上为整车的技术模块，对于汽车而言，质点模型是远远不够的，无人车是前轮转向的车，前后位置的变化是不一样的，**那么怎么去描述这种不一样呢**？

首先从刚体角度考虑，二维平面里的刚体涉及到XY和θ，也就是以车后轴中心作为XY坐标原点时车身的朝向heading。因为无人车运动模型还多了一个转向的变量，多了一个自由度，刚体模型也不够。

可以将汽车运动模型简化为自行车模型，将四轮抽象成两个轮子，前轮中心和后轮中心的运动方向和自行车一样。

车辆在垂直方向的运动被忽略掉，用一个二维平面上的运动物体来描述车辆的运动模型。自行车运动的时候具有以下特点，旋转车头的时候，前轮和后轮都围绕一个中心点转动，并且后轮的转向半径 $\frac{1}{K}$ 与方向盘转动角度 $w$ 满足以下关系，其中 $L$ 为前轮中心和后轮中心的距离。
$$
k = \frac{tan(w)}{L}
$$
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815171957427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

在实际的自行车运动模型中，后轴中心是沿着一条平滑的轨迹运行，该轨迹对应的曲率κ 表示调整方向盘的度数，如果为正，表示向左转，反之则向右转。因此，自行车运动模型可以用X，Y，θ，κ 还有速度ν来表示。

**那么沿着这样的轨迹运动时，如何去估计障碍物的距离呢？**

> 曲线坐标系

1. 曲线坐标系SL
   * ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815172347310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)
2. SL坐标系到XY坐标系的投影
   * ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815172319203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)
3. XY坐标系到SL坐标系的投影
   * ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815172524536.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

在自动驾驶中，我们将环境抽象成 SL 坐标系，在此坐标系下的曲线光滑度是有要求的。

* 曲线本身要平滑
* 其曲率也要满足平滑的特性

## 对轨迹进行平滑处理

> 生成一条光滑的曲线，涉及到
>
> 1. 目标
> 2. 工具
>
> 最简单的方法定义平滑就是最短路径，但是路径最短还不能保证平滑性，因此会对其不同阶导数进行 Minimize 求解，保证导数空间的连续，这就是 Smoothing Spline 最初的思想。那么，问题的目标就明确了，定义一个函数，能够最小化它的类似三阶导平滑性。
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815173440717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)
>
> Smoothing Spline 具有一些特殊的性质，在给定边界的条件下，它是一个多项式，可以找到最优解。但是它的 Boundary Constraint 只考虑了起点和终点，如果中间有障碍物就不是最优解。这种情况下可以使用 Piecewise Polynomial（分段多项式）来处理。

### 多项式

在轨迹上以等距离的方式随机选择一些点，然后用高阶多项插值的方式来近似表示轨迹，对多项式进行优化。但是高阶多项式不能用于平滑，因为高阶的多项式抖动太大，没有办法控制幅度，这就是常说的龙格现象。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815173102845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

### Bezier Spline

由一系列控制点定义的，例如$P_{0}$到  $P_{n}$ ，其中 n 代表曲线的阶数。

下述分别给出1阶、2 阶、3 阶 Bezier Spline 曲线的表示形式。通过对它们做平滑，得到平滑的曲线，例如二阶平滑保证曲线的曲率平滑。但是这种方法的缺点是，除了起始点和终点，其它控制点不能保证会被得到的曲线经过。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815173313447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

### Spline 2D

一个 Piecewise Polynomial 是一维的函数，描述二维曲线是不够的，这时候就有一个 Spline 2D，假设我们把曲线分成 N 截，每节曲线段它的 X 坐标是一个 Polynomial ，Y 坐标也是一个 Polynomial。

用 5 阶多项式来表示 X 和Y，称之为 Quintic Spline（五次样条），每一节都是这样的函数。

这种表示有一个很好的特性，就是目标函数具有旋转不变性。怎么让曲线足够平滑？

我们让它在 X 坐标上的变化率，也就是三阶导的平方是最小的，Y 上的变化率三阶导也是最小的，代价函数就是这两个变化率的和。代价函数的求解就是一个二次规划问题，我把这种 Loss Function 定义成这种形式是因为平方的积分能够给计算带来便利。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815173649901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

前面说的是用一节一节的线段来保证曲线是光滑的，在线段内部用一个二维的 Polynomial 表示，在内部是 N 阶可导的，但是如何保证节点处是平滑的呢？这个叫做端点约束条件，需要保证 X 和 Y 方向的倒数是相等的，一般要求到三阶导都是相等的，包括它的 X，Y 点的值也完全相等，此时就能保证三阶导连续。

### Spiral Path

还有一种方式叫做螺旋曲线，它通过一个极坐标形式定义，比如说沿着一条曲线，如果一个点 S 的曲率是知道的，假设它的原点在 （0，0）的位置，可以唯一定义出一条经过 S 的曲线，也就是 Spiral Path 。那么可以让 Spiral Path 满足起点、终点约束条件生成一条螺旋曲线。

### Spiral Path 和 Spline 2D 的区别

任何的曲线在足够密的时候都可以用Piecewise Spiral path 或者是 Piecewise Polynomial 表示。但是它们的出发点不一样，Polynomial 计算很快很简单，Spline 2D 是一个凸空间里面生成一个 Spline 曲线。Spiral Path 是从 Configuration Space 出发。理论上来讲，螺旋曲线生成的线是要比 Spline 更好处理，对一些极端情况处理更好。

# Optimization Inside Motion Planning

约束问题：

1. 目标函数的定义，目标函数比较清晰，对于后面的求解更有帮助；
2. 约束，比如路网约束、交规、动态约束等；
3. 约束问题的优化，比如动态规划、二次规划等。

## 动态规划

动态规划通过类似于有限元的方式，把问题从连续空间抽象成离散空间，然后在离散空间中进行优化。虽然这种方法可以逼近连续空间中的最优解，但是计算复杂度很高。针对计算时间长的问题，可以使用牛顿方法进行优化，它的收敛次数是指数平方，也叫二次收敛。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815174140546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

## 二次规划

二次规划算法的本质是牛顿法的 Taylor 展开，但是它的求解过程涉及更复杂的情况。因为二次规划方法并不一定是处理一维问题，可能涉及更高阶求导。在实践中，二阶导数基本可以满足问题需求。

然而，牛顿法要求 locally convex 才能保证收敛，也就是导数是严格单调递增的。但是一般函数并没有这样的特性，动态规划或二次规划都无法获得全局最优解。为了解决这样的问题，通常使用启发式搜索方法。

首先通过动态规划方式对整个问题有一个粗浅的认识，然后通过二次规划进行细化。这种启发式搜索方法也是目前百度 Apollo 的 EM 算法的核心思想。这种方法和人开车的过程是一样的，通常驾驶员会先形成一个大概的指导思想，指明往什么方向开，然后再规划一条最优路径。

**一般来说**，二次规划问题会写成一个二次函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815174253676.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

其中，X 是向量参数，Q 是一个对称的正定矩阵，$c^Tx$  是偏差项。对于这种没有约束的二次规划问题，只需要求导数等于0的那个点，使得 Qx=−C ，即可求解二次规划问题。这是一个线性方程组，它的求解速度是 O(N3)。

**对于带约束的二次规划问题，较为复杂**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815174441377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

这种情况可以有很多种解法，其中一种把限制条件放到上面的式子中，通过换元，变成一个全新的 QP 问题求解，但是这种方法很慢。另一种方法是 Lagrangian method ，通过增加松弛变量的方式去掉约束条件，变成一个可以解决的问题。

对于不等式的约束条件，如何去求解呢？可以使用 active set method，其主要出发点是最后解，可能落到边界上，如果真的是边界最优，不等式约束就可以转化为等式约束问题求解。

## Conclusion

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081517461815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

总的来说，对于求解非线性优化问题（自动驾驶中的规划基本都是非线性的），通常就是用启发式方法来求解。先用动态规划给出一个粗略解，给出一个凸空间。然后用二次规划方法在凸空间里去寻找最优解。

# Understand More on the MP Difficulty

> 前面我们提到了目标函数、约束条件、求解方法三种优化问题，在实际过程中无人车如何抽象约束呢？
>
> 在无人车场景中，有三类约束，第一个叫做 Rraffic Regulation，第二个是 Decisions，第三个是 Best Trajectory 。这些限制又分为硬限制和软限制，例如交通规则属于硬性限制。

## EM

> 在已知部分相关变量的情况下，估计未知变量的迭代技术

1. 初始化分布参数
2. 重复直到收敛
   * E步骤：根据隐含数据的假设值，给出当前的参数的极大似然估计；
   * M步骤：重新给出未知变量的期望估计，应用于缺失值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815174821662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

对换道和继续在本车道行驶分别规划出一条轨迹，只有换道之后的 Trajectory 要比本车道的 Trajectory 好的情况下才换道。在 Apollo 的 EM planner中，决定哪个道比较好的模块叫做 Reference Line Decider，中间的并行模块是通过 Path Speed Iterative 的方式并行实现的。

## 优化决策问题

> 是一个3D optimizatiion问题，包含三个维度，需要生成SLT。
>
> 三维空间优化方法：
>
> 1. 离散化方式
> 2. Expectation Maximization
>    * 降维处理，先在一个维度上进行优化，然后在优化的基础上再对其他维度进行优化，并持续迭代以获得局部最优解

对于无人车，Apollo 上的 EM planner 对 Path-Speed 进行迭代优化。

首先，生成一条 Optimal Path ，在最优路径的基础上生成 Optimal Speed Profile 。在下一个迭代周期，在优化后的 Speed 的基础上，进一步优化 Path，依次类推。它分了四步走，其中分为两步 E step 和 M step 。这种算法的缺点是不一定能收敛到全局最优解。

**优化问题：**

Objective Functional、Constraint、Solver。目标函数是一些关键特征的线性组合。约束主要包括交通灯、碰撞以及动态需求等。优化求解方法的目的是找到最佳路径，包括前面讲的动态规划+二次规划的启发式方法。

## 非线性优化问题

> 1. 动态规划
>    * 先找一个粗略解
> 2. 二次规划
>    * 从粗略解出发，找到一个最优解

以路径规划为例，假设前方有一个障碍物，首先做出从左边还是右边的避让决策，然后通过 QP 生成一条平滑的曲线去避让障碍物。对于速度而言，先通过动态规划的方式给出一个粗略的解，然后再通过二次规划的方式给出一个更平滑的解。

## 逆行

首先根据当前 Speed Profile 去估计当前逆行障碍物的位置，然后再修正 Path，根据修正之后的 Path 再来处理 Speed，例如需要减速。减速之后，估计需要重新改变路径，依此类推，直到得到理想的规划轨迹。

# Reinforcement Learning And Data Driven Approaches

决策问题通常用 POMDP 加上一些机器学习的技术来解决。

**解决规划问题的方法：**

1. 数据闭环
   * 数据驱动是在基于规则的闭环里面的小闭环。Rule Based 的方法可以对遇到的新案例，很快给出解决方案。
2. 基于规则的方法
   * 在基于规则的方法的基础上，对问题形成一定的认识，通过把问题抽象成更加通用的问题，定义目标函数来进一步优化问题。

但是解决规划问题，不是直接Data Driven上手，而是通过Rule Based基本规则开始。

有些时候环境感知并不是完全感知的，有些hidden的状态并不是完全能够知道的。

Data Driven只是加速总结，让系统变得更快。