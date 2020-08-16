# Apollo简介

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816154131415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# 本机演示实战

> Apollo的代码结构包括Docker和Docs（主要放置一些文档）、Modules（核心模块算法都在该文件夹下）以及Scripts和Tools等。

**数据流转过程：**

1. 通过高精地图和定位获得车辆周边的场景信息；
2. 通过感知模块侦测道路上的障碍物，即一些动态信息，比如旁边的车、行人、自行车等等；
3. 将感知的信息传递给Prediction，预测感知障碍物的运行轨迹；
4. 将预测结果包装再传给Planning模块。Planning根据障碍物和周边静态的情况，比如有哪些车道可选，去规划路线。路线规划完成后，将生成的轨迹传到Control模块；
5. Control模块通过Can总线协议跟车辆交互，例如应该打多大角度的方向盘，车辆现在的加速度是多少，当前是应该踩刹车还是继续跟车等。同时也会从Can总线协议层面了解到车辆本身的信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816154452854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>Apollo 主要模块之间的关系</center>

在3.0版本， 我们升级了系统里一些安全相关的模块，当发生紧急情况的时候可以直接利用熔断机制跳开PNC，直接对车辆下发刹车指令等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816154615353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

**演示环境实践：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816154714671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

最后在浏览器打开本地访问8888端口，即为Apollo

# 车辆与循迹驾驶能力实战

> 指车辆按照录制好的轨迹进行自动驾驶，其涉及到自动驾驶中最基本的地盘线控能力、定位能力、控制能力，最自动驾驶系统的一个最小子集。

在搭建自动驾驶车辆的软件、硬件环境以后，通常采用循迹测试进行验证

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816154859858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

循迹测试涉及最底下的几个模块，只需要定位、控制以及Canbus这三个模块，是Apollo的最小子集，通过循迹可以验证车的线控能力以及模块的整体集成能力。

首先在硬件上，我们需要一辆线控车辆、一个工控机以及惯导系统GPS和IMU，如果大家使用的是参考硬件搭建的车辆，不需要进行适配，可以直接进行验证。如果你不是用参考车辆来做这件事，需要做以下几步：

1. 实现一个适配器层
   * 实现新车控制器
   * 实现新消息管理器
   * 在工厂类中注册新车
   * 更新配置文件canbus/conf/canbus_conf. pb. txt
2. Can卡管理
   * 实现新can卡类’canclient’
   * 在工厂类’canclientfactory’中注册新can卡
   * 更新配置文件canbus/proto/can_card_parameter. proto
3. 控制模块
   * 创建一个控制器
   * 在’control_config’文件，为新控制器添加配置
   * 注册新控制器

## 定位

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816155954375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

**Apollo定位方式：**

* PTK定位方式
  * 基于基站的方式
* MSF（多传感器融合）
  * 利用LiDAR的3D点云来做匹配认证

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816160107269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# 障碍物感知和路径规划能力实战

> 环境感知在自动驾驶汽车应用中心占据了核心地位。一辆车要实现自动驾驶，障碍物感知是最基础也是最核心的功能。

Apollo2.0版本在1.0版本上有很大升级，主要是增加了感知和规划模块。

* 感知的核心是进行障碍物的识别、分类、语义分析和障碍物跟踪。
  * ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081616024928.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

* 规划的目的是告诉汽车按照什么样的一条路径行驶。
  * ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816160315946.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

在根据教程搭建了具备感知和规划能力的平台之后，开发者更希望根据自己的场景进行深度定制。为加速研发过程，百度提出了“云+端”的研发迭代模式，所谓的“云”大家都能理解， “端”指的是车辆端。

在运行整个流程时，所积累的海量数据，时通过云端进行传递的。当然对于传递的数据，可以分为以下几个类型：

* 原始数据
  * 各种传感器、车辆、驾驶员行为等，数据种类繁多，维度不同，数据量大，而且大多是非结构化数据，对于传输、处理、存储提出了非常大的挑战。
* 标注数据
  * 视觉的2D障碍物数据、红绿灯数据、3D点云数据等。
* 逻辑数据
  * 包括完美感知，环境的抽象以及车辆动力学模型等
* 仿真数据
  * 包括参数模糊化数据；三维重建数据等

Apollo建立了一个数据平台，对数据的采集、存储、使用进行管理。

1. 我们通过data recorder根据按照预先定义的格式生成数据，利用云端的传输机制将数据快速传递到云端。

2. 构建了一个自动驾驶数据仓库，将海量数据成体系地组织在一起，可以快速搜索，灵活使用。

3. 云端拥有基于异构系统的自动驾驶计算平台，提供强大的计算能力。

**开放资源数据集：**

* 2D红绿灯，用来识别交叉路口红绿灯数据，可以用做训练、测试和验证。

* 2D障碍物，比如车辆、行人、自行车，还有其他未知类别的图像数据。

* 3D障碍物，其实是激光雷达点云。

* 端到端的数据，提供适合end-to-end模块的数据。

* 场景解析，像素级的语义标注，比如车辆、背景、交通指示牌、障碍物等，可以用来做整体环境的识别。

* 障碍物预测，用来训练预测算法的数据集。

**开发流程：**

* 开发流程从本地开始，

* 本地开发之后可以通过Docker镜像传到云端。

* 然后在云端的计算环境中，通过调度云端的计算资源去训练算法，通过数据接口层调用六大类数据进行训练。

* 之后可以通过数据集验证算法的效果。

* 依此不断迭代，

* 直接在云端完成算法训练，提升整体的效率

# 仿真验证实战

> 推荐搭建一个ova镜像的虚拟机进行使用，开发环境的搭建

**数据集：**

* 仿真场景数据
  * 仿真需要使用的数据集
* 标注数据
  * 做深度学习感知算法训练的
* 演示数据
  * 在PC上搭建一个设备时可以做演示
* 产品服务
  * 负责上传数据的标定

**标注数据：**

* 激光点云障碍物检测分类
* 红绿灯检测
* road hackers
* 基于图像的障碍物检测分类
* 障碍物轨迹预测
* 场景解析