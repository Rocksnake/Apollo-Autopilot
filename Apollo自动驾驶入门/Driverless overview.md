## Levels of Driving Automation

There are six levels of driving automation from zero to five:

* 0: Basic level
* 1: Driver Assistance
  * Driver must remain fully engaged
* 2: Partial Automation
  * Automatic Cruise Control / Lane Keeping
* 3: Conditional Automation
  * Human take over whenever necessary
* 4: No Human Interference
  * Without Steering Wheel, Throttle or Brake
  * Restricted in Geofence
  * Outside of Geofence, the vehicles either unable to operate autonomously or unable to operate at all
* 5: Full Automation

## How a self-driving car works

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715110356913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

We use **computer vision** and **sensor fusion** to obtain a rich picture of our position in this world, use **localization** to determine our precise position in this world, and then use **path planning** to draw a path through this world to reach our destination, Then, by turning the steering wheel, opening the accelerator, stepping on the brake, driving along the track, and finally moving the vehicle. This is the principal of self-driving.

### Reference vehicle

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715111536877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)