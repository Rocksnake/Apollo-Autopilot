#　自动驾驶架构介绍

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816150012479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

## 最底层的车辆平台

> 底层车辆平台执行Apollo无人驾驶平台生成的车辆控制指令

为了能够运行Apollo生成的指令，车辆必须是线控的，例如可以接受一定的指令，比如换挡、加减速、转向，完成对应的操作。

## 传感器层

> 集成各种传感器对汽车周围环境进行感知，包括GPS、IMU、相机、激光雷达、毫米波雷达、超声波雷达等。

无人驾驶系统对算力的要求非常高，所以在Apollo上安装了一台高性能工控机（IPC）机。

* GPS/IMU主要是用于自定位
* 相机的功能主要是做红绿灯识别
* 主传感器激光雷达主要用来感知车辆周围环境
* 超声波雷达主要用来做五米范围之内的障碍物的检测；HMI是对车辆发指令的一些设备，例如平板。

## 核心的软件平台

1. RTOS实时操作系统【下面】
   * Apollo 使用打补丁的方式来实现实时的效果
2. Runtime Framework【中间】
   * 用ROS，主要是为上层的模块提供数据层支持
3. Apollo各个功能模块实现部分【上面】
   * 包括地图引擎、定位、感知、规划、监管、控制、端到端以及HMI

## 云服务层

> 提供了高精地图服务、模拟仿真、Data Platform、安全和更新、DuerOS等。

对于高精度地图，在中国，个人并不具备制定高精地图的资质和能力，因为政府要求这些数据不能在网络上传播。因此，Apollo直接将制作好的高精地图以云服务的方式对外开放。

仿真主要用来对自动驾驶的相关算法进行验证。Data Platform开放了红绿灯数据、一些典型的障碍物数据、像素级的标注数据。

# 平台的快速入门

> Apollo的快速入门方法，包括编译、高精度地图和实时相对地图、一些调试工具以及新加入的计算单元和模块

## Docker

> 一种容器的技术，它在是Linux内核的基础上做了一些轻量级和隔离机制的优化，让环境更小，部署起来更快

利用Docker可以使整个工程的安装更加简单。Docker镜像通常是一个配置好的运行环境，包括依赖的第三方库等，使得用户不需要对环境编译做过多复杂的操作。

**E.g.：**

在Release版本中，Apollo各个模块是一个已经编译好的二进制文件，可以直接运行；如果是开发版本，通常已经加载了所需的第三方库，用户只需要执行对应的编译指令。

## 硬件接入

1. 原始的UDP( User Data Packet，用户数据包 )
2. 做一个ROS Driver方法，把驱动编译到Apollo里面
3. 把数据发布出来

**Q:**

**如何使用不同于参考硬件的传感器？**

**A:**

> 通过两个例子说明

① 如何用一个新型号的Camera，假设是USB接口的相机？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816151030447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

最下层是相机硬件；往上一层是一个标准的底层驱动，即Video for Linux driver；再上一层是一个ROS Driver，最上层是Apollo可以接收到的内容。要使用该相机，主要的工作是底层硬件的解析，使得Apollo可以接收到相应的数据。

② 激光雷达

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816151633487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

硬件通常基于内核Socket的方式把数据传输到PC，PC端做一些数据处理之后发布的对应消息类型。对于激光雷达而言，发布的是Pointcloud 消息类型，该消息将被最上层的Apollo感知模块接收如下所示：右侧给出了ROS Driver如何解析UDP数据包的过程。

## 编译

1. 在Ubuntu环境下进行操作
2. 进入Docker，拉取Apollo镜像，并以此镜像创建容器
3. 进入创建的容器，编译Apollo源码

编译结束之后可以做RTK循迹测试。循迹比较简单，它包含两个文件，核心就是一个Record，用来录制轨迹的信息，也就是一些GPS点；另外记录车辆底盘返回的速度信息、加速度信息、曲率、朝向等。RTK循迹测试就是把车辆底盘发出的这些主题和定位输出进行融合。

## 高精度地图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816151838679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

<center>实际地图与相对地图</center>

实时相对地图是车辆通过传感器来感知车身周围环境，可以帮助开放者更友好、方便运行Apollo。

## 工具链

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816152005284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

此外还提供了DBC文件转换工具、Teleop、主题监控工具、配置工具等。DBC转换工具解析车辆DBC文件，生成对应的Protobuf。Teleop工具可以通过键盘控制的方式实现车辆的信号发布。

主题监控工具可以同时需要监听多个ROS topic。Configuration工具明确标识出来修改了哪些字段。另外，Apollo还提供了面向Rosbag的一些工具，包括分析规划模块、驱动以及统计信息等。

交通灯模拟工具可以通过脚本的方式控制地图里面的红绿灯变化情况，对车辆进行测试

## 模拟与Dreamview

> 整个Apollo项目可视化的一个模块，基于该模块，开发者可以在没有车和传感器的情况下使用Apollo各个软件模块

# 安装过程概述

## 安装步骤

1. 安装基础环境
2. 拉取Docker镜像并创建容器
3. 进入容器编译源码

① 首先是安装git，因为Apollo代码是托管在github平台上，所以需要git工具，然后使用git将Apollo源码克隆到本地。下载源码后，还要安装Docker环境，可以使用Apollo提供的脚本安装，也可以在官网安装。

② 环境安装完成后，运用官方提供的脚本拉去Apollo Docker镜像文件，运行dev_start.sh-C命令。其中-C选项表示使用中国服务器进行加速。在拉取成功之后，该脚本会基于镜像创建一个容器Container。

③ 接下来是对Apollo的操作，如果没有编译，需要先使用apollo.sh脚本进行编译。它有很多编译选项，默认的是Build和OP。还可以选择面向GPU编译。RS是对RB速腾聚创的激光雷达进行编译，USB Camera是对Camera的编译，这几种编译方式所涉及的类不同，所以使用的编译方式也不尽相同。

④ 编译完成之后，需要对Apollo各个模块进行调试，我们会在每个版本发布的时候给出对应的Rosbag数据包，方便去做验证。比如说Apollo 1.0提供了一个循迹数据包，2.0时发布了一个激光点云的数据包。

⑤ 我们可以启动bootstrap.sh脚本，对Apollo的bag进行回放，看一下效果。这是一个引导脚本，它做了以下事情，启动进程守护工具Supervisor，假如有进程出现不可预知的异常，这些进程通常会挂掉，经过配置后Supervisor可以保证在进程挂掉后，会将该进程拉进来。然后启动Roscore、voice_dectore和Dreamviewer。

⑥ 对于Apollo平台，很多的模块都被启动，交由Supervisor进程进行监控，包括Can Bus、 激光雷达、控制模块、GPS、Mobileeye、NG等模块。

⑦ 在运行完bootstrap.sh脚本之后，在浏览器地址栏输入localhost：8888查看Demo的演示效果。Demo加载bag对应的数据，包括车辆的数据、障碍物数据、绿色障碍物ID、速度、形态。车在运行过程中需要查看的不仅仅是仿真出来的场景，还要看一些跟Planner、控制相关的信息。

## 使用仿真平台

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200816152857676.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# 仿真平台使用

Azure是一种灵活和支持互操作的平台，它可以被用来创建云中运行的应用或者通过基于云的特性来加强现有应用。它开放式的架构给开发者提供了Web应用、互联设备的应用、个人电脑、服务器、或者提供最优在线复杂解决方案的选择。

> 是微软的一个仿真平台Azure，它不需要本地部署

在Apollo的Github账号上可看见上图所示的两个状态，左侧的Build用来做持续集成。

为了简化验证，团队会把已经编辑好的测试运例用来测试开发者提交的代码是否正确，以此来判断开发者的代码对目前的Master的分支是否有影响。

* Build提供了对开发者代码验证的一种渠道。
* Simulation主要用来验证代码的鲁棒性。

Apollo团队在微软的Azure仿真平台上部署了很多场景，拿最新的代码去在这些场景下进行测试和验证，看相应模块在这些场景的执行情况，最终得到代码的鲁棒性报告。

该仿真平台的地址是azure. apollo. auto。在该仿真平台运行自己的代码是不需要进行本地编译的。

**使用流程：**

1. 克隆Apollo在Github上的代码
2. 在本地对相应的模块进行修改
3. 修改之后将代码提交到自己在Github的Apollo仓库中，可以是Master分支也可以是新建的分支
4. 在微软的Azure仿真平台选择目标场景对更新后的代码进行验证。

运行之后会拿到一个报告，表示修改的代码再不同场景下的执行情况。

其中第一列的Scenario是一些场景，在仿真平台中，我们会把一段很长的路切割成很多的场景，比如有左转、左转有行人、有行人横插等。后面几列是对应场景的状态描述，Run Status表示场景的运行状态，如果后面的指标有一个失败，那么Run Status就是失败的。

具体衡量的指标有：碰撞检测、速度校验、On Road检测、Red-Light检测（是否有闯红灯的情况）、ARW检测（是否成功到达目的地）、Hard Break（急刹车）、加速度（它是影响体感的一个指标）。