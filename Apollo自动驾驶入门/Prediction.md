# Prediction

How Apollo predicts trajectories for moving objects based on both their state and the location of our own self-driving?

> * real time
>
> > want the latency to be as small as possible.
>
> * accuracy
>
> > as accurate as possible so that autonomous vehicle makes good decisions.

## Prediction Approaches

* model-based prediction
  * when facing some special scenario, we actually construct all possible candidate predictive models.

* data-driven prediction
  * use a machine learning to train a model based on observations, once the machine learning model is trained, we can use it in real world.

![image.png](https://i.loli.net/2020/07/17/mDfMlWL1xXkhQzB.png)

For Example:

1. Determining maximum safe turning speed on a wet road by Model-Based.
2. Predicting the behavior of an unidentified object sitting on the road by Data-Driven.
3. Predicting the behavior of a vehicle on a two lane highway in light traffic by Model-Based.

## Lane-Sequence-Based Prediction

> a model-driven approach

* divide the scenario into mutiple segments
* group behaviors into a limited set of patterns
* describe these patterns as sequences of lane segments
* describe the target vehicle’s motion based on the sequenecs

### Pradict Target Lane

* Using lane sequences frameworks to generate trajectories for objects on the road.

* List all trajectories.
* simplify the prediction preblem into selection problem.
* Choose the lane sequences that the vehicle is most likely to take. 

![image.png](https://i.loli.net/2020/07/17/tOnYXFDzKSIg3yR.png)

![image.png](https://i.loli.net/2020/07/17/hsIy8RDq2mSAdpT.png)

## RNN(Recurrent Neural Networks)

> prediction that takes special avantage of time-series data.

The fundamental structure of RNNs

![image.png](https://i.loli.net/2020/07/17/HGPeMvSAdW73O2o.png)

## Application of RNN

* for lane sequences
* for the associated object states
* concatenates the outputs of these two RNNs and feed them into one neural network.

![image.png](https://i.loli.net/2020/07/17/yWnqFkjXK6CrHIT.png)

## Trajectory Generation

> The final step of predction

Between two points, there are infinite candidate trajectories for an object to travel.

* set constraints that will eliminate the most of candidate trajectories.
* notice the position and heading of vehicle at both points, then fit a polynomial model using these two conditions.

## Recap

* how to transform complicated vehicle movements into lane transition sequences
* how to use existing observations represented by lane sequences to train neural networks, to make predictions
* incorporated lane sequence prediction with vehicle physics to generate trajectories for each object 