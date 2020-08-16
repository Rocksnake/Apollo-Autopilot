>  控制技术在目前无人车方案中的限制以及未来的发展，控制技术与无人车其他技术模块的联动。

> 控制模块根据预测的轨迹和估计的车辆状态向油门、刹车或转向扭矩发送适当的命令。控制模块使汽车尽可能接近计划的轨迹。控制器参数可以通过最小化理想状态和观测状态之间的误差函数(偏差)来估计。

# Control in Apollo - 1

* 通用控制理论及其在Apollo自动驾驶平台上的应用
* 功能的限制和未来的趋势
* 类似的原理怎样应用于不同的模块

> Apollo 架构包含了感知、定位、决策、控制、通信模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816121058621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

可以看到控制模块的信息是规划模块（planning）和自反馈阶段信息（如localization和HD Map），它们会经过一个交互的过程，将相互之间的信息进行交流和决策。

## 控制模块

* 预处理
  * 对输入信号检查过滤不正常信息
  * 紧急处理
  * 做滤波操作，如信号的平滑
* 控制器
  * 模型建立、系统识别和分析
  * 控制器/观察器设计
  * 参数调优
* 后处理
  * 将信号发送给执行器，包括限制的处理以及信号的滤波。

> 控制主要是为了弥补数学模型和物理世界执行之间的不一致性。

**需求：**

1. 稳定性，在所有场景下的车辆行为稳定和安全
2. 稳定状态的行为，减少或者消除规划和实际车辆行为的差别
3. 瞬时状态的行为

可以把性能评测具体化，映射到三域：时域、频域以及离散域（discrete domain）

时域是指输出在时间上应该满足的要求，其衡量标准是steady state gain、rising time、setting time、overshoot和undershoot。其具体意义如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816141701980.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

在频域空间，X轴是输入频率，Y轴是输出跟输入的比例，联系状态下输出和输入的比例应该为1.

除了时域和频域的要求，还需要满足discrete domain的要求

对系统来说，在time domain跟frequency domain中的系统需求是可以等价转换的。系统在时域的要求、响应、数学表达跟频域是可以相互对应的。

现如今最简单的控制器就是PID控制器，即比例、积分、微分控制，它是一个model free的控制方法，就是说其控制具有通用性。其原理如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816141600792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# Control in Apollo - 2

> 上面提到的PID控制，虽然起到了调节稳定性的作用，但还是很难做到精准控制

Apollo 基于模型控制方法，就建模、系统辨识、控制器设计和参数调优。方面设计新的控制方式。

## Modeling

* 分析建模
* 拟合建模

> 一个模型主要由各种属性表示，主要包括描述输入输出的数量、线性或者非线性、离散还是连续等等。

控制模块中的模型，通常包括动力学模型和运动学模型。运动学模型是一种几何模型，感知、预测讨论的模型模型则以运动学模型为主。而在控制模块中，更多考虑动力学模型，实际上，运动学模型是动力学模型的子集。

在自动驾驶中，Dynamic model以Kinematics model为初始模型，将环境等参数设置到Kinematics model中，把车看作质点进行分析。Dynamic model将车按车轮等部分分开进行约束或者系统补偿。

**E.g.：就两个几何模型进行分析：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816142315518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>左边是综合移动机器人控制模型，右边是自行车模型</center>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816142430334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>考虑了力矩和扭矩平衡</center>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816142530203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>考虑了缸体力矩和扭矩公式，总体满足牛二律</center>

进一步，在假设纵向速度为0的情况下，我们可以对横向方程进行线性化。

其过程需要基于一些假设。所以在做完控制之后，我们还需要检查这些假设是否合理或者是否会造成很大的误差。

在控制器实现过程中，通常会将ODE或者PDE方程进行处理，转化为矩阵计算的形式。虽然在数学表达形式上不一致，但是其物理含义保持不变，

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816142718793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>在状态空间表示中还会给出一些状态量的标识，包括输入量。</center>

## 系统辨识度

* 白盒
* 灰盒
* 黑盒

|   方法   |                             原理                             |
| :------: | :----------------------------------------------------------: |
| 白盒方法 | 基于第一原理（如牛顿定律）的模型结构，可以由测量数据估计模型参数 |
| 灰盒方法 | 用于只有部分模型结构可知，通过数据重建的方法来获取模型的其他部分的方法 |
| 黑盒方法 | 指模型结构和参数都在未知的情况下，只能通过输入输出数据来估计的方法 |

## 控制器设计

1. 滤波器设计
   * 线性与非线性
   * 数字信号与模拟信号
   * 离散与连续
2. 控制器设计
3. 观察器设计

系统在频域里面需要满足某些性能要求，滤波器通常会对频域信号进行处理。当然我们会根据实现的方式不同，将器分为高斯滤波、卡尔曼滤波、贝叶斯滤波等。它们的作用通常是预测和跟踪。

# Control in Apollo - 3

控制器类型：

* 开环控制
* 前馈控制
* 后馈控制

## 前馈环控制

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816143244678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

前馈环控制器的主要控制策略

* Optimal Control（优化控制）
* Adaptive Control（自适应控制）
* Robust Control（鲁棒性控制）

### 优化控制

> 在特定情况下，找出一个使系统满足特定条件的优化标准。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816143401658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

### 自适应控制

> 针对控制系统中参数多变或者初始值不确定的控制方法。最简单的方法就是根据输入使用swith的方式，根据输入或者gain选择不同的控制算法。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816143517277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

### 鲁棒控制

> 解决如何确定模型的正确性问题。它主要用来处理模型的非确定性，是一种在已知模型错误边界的情况下，设计一个性能不错而且稳定的控制器方法。最简单的鲁棒性控制器是LQR/LTR控制器，也是一个二次线性调节器。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816143556460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

## 离散化

> 在尽可能的保存连续空间信息的情况下，把连续空间的问题转换为离散空间的描述，使得计算机能够更好地处理。

<font color="red">注意：离散化跟Digital Stability是相关的，如果采样不够好，会丢失很多信息使得系统不稳定。</font>

### 方法

> 都是把数字信号转换为模拟输入/输出信号

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081614382546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>左边是时域表达式，右边是离散化的常用表达形式，最后一列是收敛速率</center>

> 收敛速率：在一定情况下，数字控制在给定时间下是可以保证收敛的。

## 控制器设计的其他方面

* Deadzone

  * 执行器的一些特性引起的

    E.g.：

    * 汽车的油门，可能给油在0%到15%之间都是不会使汽车前行的，这个时候反应在图上的就是一条平行的线段，我们称之为Deadzone。在控制器设计的时候需要对这部分进行补偿设计。
    * ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816144259995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

* 饱和和抗饱和

  * 出于对执行器的特性的考虑，通常一个执行器是有上限和下限的

    E.g.：

    * 把输出值做一个限制，使得输出在执行器的上下限范围内。如果不进行饱和处理，在输出100%的情况下突然转换状态，收敛到最终值可能需要很长的时间。
    * ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816144320191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

## 发展趋势

* 数据驱动
* 结合轨迹生成
* 结合预测的控制
* 基于模型的增强的学习控制方法

## 案例

在Apollo中，控制的工程应用主要有两个方面，一个是系统识别，使用的是自动标定方法。另一个是安全策略。

安全策略的考虑主要是基于控制是否与底层交流的最后一个模块，所以有很多的安全策略需在控制层面完成。安全信息可分为两个部分：上游信息（Planner发出）+下层反馈信息。如果上游Planning信息丢失、延时、未更新，控制系统需要做出诸如Emergency Stop、缓行之类的决策。类似的，由于接触不稳或者其它因素，导致控制指令没有执行，控制器也需要做一些安全策略的考虑。