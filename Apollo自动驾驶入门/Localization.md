# Localization

> Locaization is how an autonomous vehicle knows exactly where it is in the world.

* GNSS RTK
* Inertial navigation
* LiDAR localization
* Visual localization

## GNSS RTK

**Triangulation**

> If you have a map that tells you where those landmarks exist in the world, then you know exactly where you are in the world.

 **GPS**

> Use satellites that are broadcasting their distance to us from space.

![image.png](https://i.loli.net/2020/07/16/awHynfsBR1otTSk.png)

The GPS receiver is not actually directly detecting the distance between you and the satellite.

**RTK**

> In order to reduce the error further, this is a Real Time Kinematic positioning. And it involves setting up several base sations on the ground.

![image.png](https://i.loli.net/2020/07/16/QpBW2lDjM7khXSv.png)

### Pros and Cons

* <font color="gree">Accurate with RTK</font>
* <font color="red">Poor Performance in Urban Area and Canyons.</font>
* <font color="red">Low Frequency Update</font>

> Since autonomous vehicles move quickly, we may need to update our position more frequently than 10 hertz or 10 updates per second.

## Inertial navigation

We can use acceleration, initial velocity and initial position to calculate the velocity of the car and the location of the car at any point in time.

**To measure acceleration**

> a sensor called a three-axis accelerometer

record measurements based on the coordinate frame of the vehicle.

**To translate measurments into the global coordinate system**

> gyroscope

<img src="https://i.loli.net/2020/07/16/PtroD3dIfqjByhA.png" alt="image.png" style="zoom:80%;" />

calculate the location of the vehicle in the global coordinate system by measuring the relative positions of the spin axis and the three external gimbals.

These two are the main components of an inertial measurement unit or IMU.

**One inportant feature of an IMU is high-frequency updating, and we also know that GPS with lower frequency. **

### Combination

|              |                      GPS                      |           IMU           |
| :----------: | :-------------------------------------------: | :---------------------: |
|  advantage   | there is no motion error increasing with time | high-frequency updating |
| disadvantage |             low update frequency              |      motion errors      |

So we can combine these two to localize a car.

## LiDAR localization

> Even combine GPS and IMU, when we traveling in the mountains or in urban canyons or worst of all in underground tunnels. Then we might go for long period of time without a GPS update.

With LiDAR we can localize a car by means of point cloud matching. By comparing detected data from the LiDAR sensors with the preexisting high-defination map, it can yields the global position and heading of the car.

**match poing clouds**

For Example: ICP, Filter, Histogram Filter, Kalman Filter

<img src="https://i.loli.net/2020/07/16/nsldIcBrzTXeVup.gif" alt="2020-07-16-11-08-40.gif"  />

In this progress we want to minimize the average distance error by rotating and translating the point clouds.

### Histogram Filter(or called SSD)

* slide the point cloud scan from the sensor across every position on the map
* at each position, calculate the differences and distances
* sum the quares of the differences
* to select, the smaller the sum, the better match

|              |                      GPS                      |           IMU           |                            LiDAR                             |
| :----------: | :-------------------------------------------: | :---------------------: | :----------------------------------------------------------: |
|  advantage   | there is no motion error increasing with time | high-frequency updating |                          robustenss                          |
| disadvantage |             low update frequency              |      motion errors      | difficulty of constructing high-definition maps and keeping them up to date |

## Visual localization

Because of cameras are cheap, plentiful and easy to use, cameras imagse are the easuest type of data to collect. But usually we don’t just use images alone to localize cars, we combine cameras images with a map and GPS to localiza cars.

![image.png](https://i.loli.net/2020/07/16/k2HGgJc5L8xU1hb.png)

We using observations, probability, and the map to identify our most likely location. This process is called a **particle filter.**

|              |                      GPS                      |           IMU           |                            LiDAR                             |                            Visual                            |
| :----------: | :-------------------------------------------: | :---------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  advantage   | there is no motion error increasing with time | high-frequency updating |                          robustenss                          |                 image data is easy to obtain                 |
| disadvantage |             low update frequency              |      motion errors      | difficulty of constructing high-definition maps and keeping them up to date | the lack of three-dimensinal information and the reliance on three-dimensinal maps |

## Multi–sensor fusion localization system

> Apollo uses a multi–sensor fusion localization system based on GPS, IMU and LiDAR

![image.png](https://i.loli.net/2020/07/16/1NGWsOUtp36Idjf.png)

Diffusion framework combines these outputs through Kalman filters.