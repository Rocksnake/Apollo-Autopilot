# 高精度地图与自动驾驶的关系

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815140058642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

> L3级别以下不需要高精地图，但在L3、L4级别，高精地图是标配。发展到L5阶段是否需要高精地图？到目前为止还不确定。在目前的情况下，如果没有高精地图，L3、L4级别的自动驾驶很难实现。

**高精地图的特征**：

1. 描述车道、车道的边界线、道路上各种交通设施和人行横道
   * 即把所有东西、所有人能看到的影响交通驾驶行为的特性全部表述出来。
2. 实时性
   * 自动驾驶完全依赖于车辆对于周围环境的处理，如果实时性达不到要求，可能在车辆行驶过程中会有各种各样的问题及危险。

> 高精地图对于感知、定位、规划、决策、仿真和安全都是不可缺少的

* 导航地图只是给驾驶员提方向性的引导。识别标志标牌、入口复杂情况、行人等都是由驾驶员来完成，地图只是引导作用。导航地图是根据人的行为习惯来设计的。

* 高精地图可以作为自动驾驶的「大脑」。「大脑」里面最主要是地图、感知、定位、预测、规划、安全。综合处理成自动驾驶车辆能接受的外部信息，并统一运行在实时的操作系统上。

**定位方案：**

> 对周围环境的感知

1. 基于点云
   * 自动驾驶车辆在路口“看”到建筑物，然后通过激光雷达能搜到点云的信息，通过点云的特征提取，然后通过复杂的组合变换、视角变换，最终通过跟周围环境的比对能得到比较准确的定位坐标。
2. 基于Camera
   * 局限大，在夜间、逆光的情况下很难达到非常好的视觉效果

自动驾驶车辆搭载的传感器类型很多，但是多具有局限性，高精度地图可以为之提供帮助，

**E.g.:** 

在高精地图里提前标注红绿灯的三维空间位置后，感知模块就可以提前做针对性检测。这样做不仅可以减少感知模块的工作量，而且可以解决Deep Learning 的部分缺陷。识别可能会有些误差，但先验之后可提高识别率。

**规划：**

> planning 模块

1. 长距离规划
2. 短距离规划

**预测**模块的作用是把其他道路参与者的可能行驶的路径轨迹和行动预测出来。

**决策**模块主要是根据规划和预测的结果决定自动驾驶车辆是跟车、超车还是在红绿灯灯前停下等决策。

**控制**模块是把决策结果分解为一系列的控制行动，然后分发给控制模块执行。

> 定位、规划、预测、决策、控制模块都与高精度地图有着密切的关系，高精度地图能使车辆决策更加准确。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081514182986.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

**高精度地图的特点**

1. 表征路面的基准全面性
2. 高精度地图要求更高的实时性
3. 高精度地图=自动驾驶地图

**高精度地图与安全模块**

1. 针对传感器的攻击
2. 针对操作系统的攻击
3. 针对控制系统的攻击
4. 针对通讯系统的攻击

**仿真**

搭建与真实场景高度一致的方针场景，协助自动驾驶测试开发工作，高精地图为仿真地图提供了最底层的基础结构，能让仿真系统更好的去模拟真实道路的场景。

# 高精度地图的采集与生产

> 高精度地图的采集与生产也是一系列非常复杂的行为，需要结合各式各样的传感器与算法

## 采集

需要用到的**传感器**有：

1. GPS
   * 空间点位置的计算原理 （通过GPS）：空间点位置是一个三维坐标，有三个变量，需要三个方程。
   * 通过观测三颗卫星与空间点位置的距离，利用三角测量法，就可以准确地得到地球上任何一点的空间位置。但三颗卫星的测量方案在实际应用中，可能会存在误差。
   * 因此，在空间点位置的计算过程中，我们经常要检测四颗或四颗以上卫星，才能实现精确的定位。
2. IMU
   * 目前 IMU （惯性测量单元）是自动驾驶汽车的标配。
   * IMU是测量三轴加速度的一个装置，通过算出积分，得到任意两帧间的相对运动。
   * IMU有高端和低端之分。高端IMU能保持较长时间的计算精确度，而低端IMU在GPS信号丢失的情况下，能够维持比较精确的时间非常短。
   * 实际工作中，由于不可避免的各种干扰因素， 如果不对该运动加以校正，IMU的误差会就随着时间的推移变得越来越大。
3. 轮速计
   * 目前，轮速计的使用非常普遍，很多汽车都配备了轮速计。
   * 轮速计受地面材质的影响很大
   * 但是由于车型差异、地面交通路况不同，就会导致轮速计统计结果的差异。

**主流制图方案：**

1. 基于激光雷达
   * 激光雷达 通过发射和接收激光光束得到两点之间的距离，因此其精确度非常高。
   * 激光雷达内部的扫描部件与光学部件，通过收集反射点与反射点发生的时间和水平角度，从而得出任意一点的空间信息和光强度。
   * 该坐标信息扫描的是某个局部，通过一定的坐标转换，能够形成一个全局的坐标系。
   * 通过将GPS、IMU和轮速计测出的数据进行融合，再运用Slam算法，对Pose进行矫正，最终才能得出一个相对精确的Pose。
   * 最后把空间信息通过激光雷达扫描出三维点，转换成一个连续的三维结构，从而实现整个空间结构的三维重建。
   * 通过扫描的激光点和GPS、IMU的测量数据综合运用，能够计算出一个 预测结果与实际结果最小差距的数值信息。
   * 但这只是我们在高精地图采集过程中一个最优化的计算模型，实际情况比这个要复杂得多
   * 虽然激光雷达采集的信息非常精确，但它采集的信息非常少，无法提供像图像那样丰富的语义信息、颜色信息。
2. Camera 和融合激光雷达
   * 通过丰富的图像信息和准确的激光雷达数据，最终得出一个非常准确的高精地图。

|     方案     |                  特点                  |
| :----------: | :------------------------------------: |
|   激光雷达   |    信息准确，但是信息较少，不够丰富    |
|   相机视觉   | 色彩丰富，信息量大，但是处理起来不准确 |
| LiDAR+Camera |    结合两者的优点，信息量丰富且准确    |

# 高精度地图的格式规范

> 对采集到的地图如何进行一个完整的表述

通过格式规范：

1. NDS
   * 一个NDS不仅包括基本导航技术数据、B公司的POI数据（即地图上的一个点，地图上每一家商家店铺都可以被称之为一个POI数据点），还支持局部更新，即使是对一个国家或者省市的相关内容进行局部更新，都十分便捷。
   * 为了方便用户，NDS还提供语音、经纬度等描述功能。
2. OpenDRIVE
   * Apollo也开发了自己的OpenDRIVE格式规范
   * 在运用OpenDRIVE格式规范表述道路时，会涉及Section、Lane、Junction、Tracking四个概念。
   * 无论车道线变少或变多，都是从中间的灰线切分。切分之后的地图分为SectionA、SectionB和SectionC三部分。
   * 一条道路可以被切分为很多个Section。按照道路车道数量变化、道路实线和虚线的变化、道路属性的变化的原则来对道路进行切分。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815143844939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# 业界高精图地图产品

## HERE HD Live Map

> 通过检测与定位约束纵向行驶信息，车道线约束横向行驶信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815144138469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

基础地图的设计、众包更新、在云计算中的映射学习、更新的地图

**HERE采集车**，集成了16线激光雷达+ Camera + RTK天线+IMU，对地图进行预先制作，在数据采集后进行数据统计，经人工识别检查后，最后更新在地图中，车端通过Sensor进行信息采集（可认为一种视觉方案），可对点、道路、标志标牌通过Feature进行提取，可有效帮助我们更快的对地图进行更新。

**原理**：不同于利用神经网络的图像处理方法，HERE利用点云分割技术对Features进行分析。在多次采集后，可将同一区域的点云补齐，但目前的图像处理方法已较为成熟。而点云技术（点云SLAM、点云分割、点云特征提取等）仍需完善发展。

**对地图的描述方式：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815144442278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

## MobileEye

**技术层次：**

1. **感知**
   * Mobileye的软件可以进行传感器融合——从摄像机传感器、雷达和激光雷达传感器中解读数据。
   * 在图像处理方面，Mobileye经验丰富，使用自己独有算法是用来检测对象，确保安全行驶和系统决策。
   * L3以下的自动驾驶不需要高精地图，但是L3以上就看你使用的是基于Lidar还是Camera的方案了。

2. **地图**
   * 自动驾驶汽车需要大量的系统冗余来处理无法预料的情况。在所有条件下，车辆相对于道路边界和交叉口的精确定位都需要高精地图。
   * Mobileye提供基于REM的框架(REM™)，它使用众包的策略。让用户能低成本地构建和快速更新高清地图。

3. **驾驶策略**
   * 一旦一辆自动驾驶汽车能够感知周围的场景并在地图上进行定位，要解决的最后一件事情，就是学习和共享人类司机的驾驶策略。
   * Mobileye声称，传感、测绘和强大的计算能力赋予了自动驾驶车辆超人的视觉和反应时间。
   * Mobileye对驾驶策略的强化学习，将提供多变量情况的分析方案，并且尽可能地逼近人类的行为和判断方式。这证明Mobileye对于复制人类的驾驶行为还是很看重的，至少把其单独地作为一个数据层去阐述处理。

**REM解决方案：**

1. 采集器（任何装有摄影机的车辆）
2. 云端
3. 自动驾驶车辆

## Google Waymo

> 运用无人车基于高精度地图的静态信息和感知器对环境的动态感知融合两者，以便更好的利用信息

谷歌将地图提供的静态环境和基于感知的动态环境（人物、车辆、道路标志）等信息结合在一起。使搭载Waymo的无人车完成对环境的感知。

谷歌同样将红绿灯感知为框体，并且将人行横道的识别放在非常重要的位置。

谷歌将根据地图提供的静态信息确定红绿灯的位置，基于感知到的红路灯状态为其打上标签（红灯禁止或者是绿灯通行），再为车辆决策提供依据，并且有蓝色的预测轨迹为车辆规划行驶路径。

## TomTom

> 基于激光雷达设备和神经网络算法对图像进行识别。

把环境的特征利用点云做出模拟的环境，以便在驾驶过程中，汽车行驶的道路能够用计算机实时处理，提高了计算能力。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815145607168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

# Apollo地图采集方案

> 百度采取的是激光雷达和Camera二者相结合的制图方案，基于Apollo的地图数据采集可以实现一键化

|  Equipment   |                  Function                  |
| :----------: | :----------------------------------------: |
| 64线激光雷达 |                采集路面信息                |
| 16线激光雷达 |             红绿灯或者高架信息             |
|     GPS      |     基于基站，利用**RTK**方式精确定位      |
|     IMU      | 检测和记录汽车行驶时的加速度，提供相关信息 |
|    长短焦    |             周围图像环境的采集             |

## RTK 方案

> 相较于GPS提供更高的精度
>
> 需要观测点：
>
> * 静态
> * 动态

观测站通过长时间在某个位置不断地进行定点观测卫星、观测计算，是一个静态的观测站点。而无人车相当于一个动态的站点，通过车辆移动监测卫星信号。

GPS在传输过程中，可能会受到多径效应、电离层大气层、反射折射等各种元素的影响。但一定范围内的不同基站，受到的影响相对一致。

基于该原理，RTK方案通过观测站之间载波信号的差分就可以得到厘米级的定位效果。

RTK方案需要基站在无遮挡的情况下，才能提供非常准确的位置。

## 地图采集

> 先决条件：
>
> 1. 传感器工作状态
>    * 绿色为正常、红色和黄色为故障
> 2. 传感器是否已被标定
>    * Camera内部参数和外部参数不一致，会导致采集的数据不准确，从而作废。
>    * 相应的，不同厂家生产的激光雷达，其参数设定也会不一致。
>    * Camera和激光雷达都需要被标定

采集过程中，无人车需要双向车道全覆盖3—5遍，最好是5遍。

如果车辆搭载64线激光雷达，那么完成地图采集目标所需要的全覆盖圈数可以减少。16线激光雷达则需要跑更多圈，才会得到更为精准有效的数据。

Apollo地图采集对车速并无明确要求，但为确保采集效果，时速低于60千米为宜。

Apollo采集路口红绿灯时使用的是Riegl传感器。所以在路口采集时，我们并不需要将车停下来进行静态扫描。这种行为本身十分危险并且违反交通法规。车载Riegl可以保持在正常行驶状态下，就能够采集到路口红绿灯的信息。

一次采集行为完成后，所有的结果会形成数据包。其中包含点云、车辆标定参数、定位结果等信息。制图过程是离线的。

# Apollo地图生产技术

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081515090857.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

**数据采集**

百度采取的是激光雷达和Camera二者相结合的制图方案。

Apollo2.5版本中，百度已经发布了其地图采集方案。该方案的基础传感器配置有：平装的64线激光雷达和16线激光雷达。

其中，64线激光雷达用于道路路面采集。由于其扫描高度比较低，还需要一个斜向上装的16线激光雷达，用于检测较高处的红绿灯、标牌等信息。其他传感器有GPS、IMU、长焦相机以及短焦相机。

**数据处理**

传感器采集到的数据分为点云和图像两大类。L4级自动驾驶汽车对地图的精度要求非常高。Apollo在制图过程中处理的数据也以点云为主。采用RTK的先决条件，即在开阔无遮挡的情况下，才能取得相对准确的信号。在城市道路中采用RTK方案，由于高楼遮挡或林荫路等场景无法避免，它们仍会对信号的稳定性产生影响。因此，我们在拿到点云之后需要对其进行拼接处理。

1. 点云拼接：采集过程中出现信号不稳定时，需借助SLAM或其他方案，对Pose进行优化，才能将点云信息拼接，并形成一个完整的点云信息。
2. 反射地图：点云拼接后，可将其压缩成可做标注、高度精确的反射地图，甚至基于反射地图来绘制高清地图。其生产过程与定位地图的制图方式一样。

**元素识别**

元素识别包括基于深度学习的元素识别和基于深度学习的点云分类。

基于点云压缩成的图像进行车道线的识别，我们可得出准确的车道线级别的道路形状特征。除此之外，我们还需要提炼道路的虚实线、黄白线、路牌标识等，来完善道路特征。通过对收集到的图像等进行深度学习，即可提炼出道路相关元素放到高精地图中。

数据采集、数据处理、元素识别三个流程是高精地图自动化的必要环节。不过，从目前来看，自动化仍无法解决所有问题，仍存在信息补齐和逻辑关联的缺陷。

* 无人驾驶车辆无法处理没有车道线的道路。这一步需要离线并用人工手段补齐相关信息。
* 涉及到逻辑信息的处理时，无人车无法判断。例如在某一路口遭遇红绿灯时，车端应该识别哪个交通信号灯，也需要人工手段关联停止线与红绿灯。

**人工验证**

人工验证的环节包括识别车道线是否正确、对信号灯、标志牌进行逻辑处理、路口虚拟道路逻辑线的生成等。

## 通过64线激光雷达采集到的点云信息加上定位信息，如何拼接得出一个完整的点云效果?

1. 全自动数据融合加工
   * 依托模式识别、深度学习、三维重建、点云信息处理等世界领先的技术，其数据自动化处理程度已达到90%，相对精度达0.1-0.2米，准确率高达95%以上。
   * 简单的说，采集到的这些每秒 10 帧左右的图像，识别和融合都是自动化的。把 GPS、点云、图像等数据叠加到一起后，将进行道路标线、路沿、路牌、交通标志等等道路元素的识别。
2. 基于深度学习的地图要素识别
   * 能否根据点云分割从中提取精确的Feature
   * 尝试从点云中提取车道线、灯杆、红绿灯。如灯杆可以用来做视觉定位的Feature
3. 人工生产验证
   * 融合底图、图像、点云数据、整合生成高精度地图数据

4. 地图成果
   * 生成包括定位地图、高精地图、路径规划地图、仿真地图在内的结果

# Apollo 高精地图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815152557606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)

**逻辑关系表述**：当前，地图中各个元素之间的关系并没有嵌入到元素的表述中，而是使用overlap来表述两个元素之间的关系。Overlap主要是用来描述两个元素的空间关系。

## 坐标系

1. UTM坐标系
   * UTM坐标系把全球分成60个区域带（Zone），每个Zone里面都是相当于Zone中心的一个局部坐标系
   * UTM坐标系描述的位置十分精确。目前，Apollo内部主要采用UTM坐标系
2. 84坐标系
   * 84坐标系是一套全球经纬度，也是高精地图里面使用的坐标系
   * 在该坐标系中，把整个地球想象成是一个椭球，地面的高度是相对于椭球面的一个偏移。高由正数表示，低由负数表示
3. Track坐标系
   * Track坐标系是基于st的，如上图所示。s是纵向，t是横向
   * 这个坐标系用来表述一个元素跟Lane之间关系，描述它位于Lane的什么位置，相对于Lane起点的偏移量是多少

## OpenDrive规范

1. Apollo OpenDRIVE把所有元素做了归类。
2. 类似于Road和Junction。路上的所有的地面标识都归属为Objects，所有的标牌都归属为Signal，并通过Overlap把它们关联起来。

**Apollo的OpenDRIVE跟标准OpenDRIVE的区别：**

* 元素形状的表达方式不同
  * 标准OpenDRIVE是基于参考线加偏移，并采用方程来描述。Apollo里面的OpenDRIVE，都是坐标点，没有采用方程的方式。
  * 采用方程方式的好处在于数据量非常小，通过三四个参数就可以描述一个非常长的线。采用坐标点的方式，数据量会稍微大一点。但是也有很多的好处。第一，用点表示对于下游的计算非常友好，不需要再重新通过线去做点的采样。
* 在道路急于转弯的地方，原始的OpenDRIVE把基于Reference Line的方式还原成点的方式，会导致道路上存在毛刺
  * 这种处理方式对于无人驾驶来说非常危险。一旦道路出现毛刺，就会导致无人驾驶车猛打方向盘，可能直接冲到路边上去。其次，Apollo对OpenDRIVE进行了元素类型的扩展。比如增加了禁停区，人行横道、减速带等元素的藐视。
* 增加了一些道路元素关系的表述
* 增加了诸如停止线与红绿灯的关联关系，中心线到边界的距离等的描述

## HDMap Engine

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200815153717331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70#pic_center)