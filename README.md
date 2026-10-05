# numpy-sensor-fusion
In this repository I will explore the main issue (noises) linked to signal acquirement and prove on a graph the effect of filters (Complementary and Kalman Filter) on these results. Starting from a sinusoidal wave to which an artificial Gaussian noise is applied, the projects proves how the statistic and predictive approach (Kalman) and the computationally inexpensive blending of sensor data (Complementary Filter) can rebuild a steady signal from a disturbed one. The signal is meant to represent the value of a certain angle theta (in the graph expressed in degree) over a "t" seconds interval. For simplicity, I have chosen a 2D case, wher we can im agine a drone flying.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The "noise" idea

No measurement in practical application is totally aligned with a "clean" model of data acquirement. The term "noise" is used to describe these casual variation.   

Note: in our simulation, a Gaussian noise will be injected into a sinusoidal wave. This kind of disturb is a mathematical model widely-used for simulation and tests:

- it closely mimics natural interferences, such as the noise produced by digital sensors or hardware fluctuations.

- being based on a normal distribution, it is highly manageable to study, analyze, and implement in software.

- its fluctuactions are simply added to the lean signal, amking it easier to implement algorythm able to eliminate it.

Central Limit Theorem

The Theorem states that the sum of a large number of independent random variables tends toward a Gaussian bell-curve distribution. In IMU, since independent noise sources sum up  together, the overall measured noise follows a Gaussian distribution. 

The Complementary Filter

A complementary filter is a quick and effective method for blending measurements from an accelerometer and a gyroscope to generate an estimate for orientation.

The filter works assigning specific weights to the measurements provided by the two sensors, so as to:

-	Smooth spikes measured form the accelerometer
  
-	Reduce the offset caused by the gyroscope drift

Accelerometer

The sensor estimates the value of theta in a 3D space computing 

theta = arctan((a_x)^2/sqrt((a_y)^2 + (a_z)^2)).

For this analysis, I assume that it is only affected from noises from the drone’s vibrations, and any static bias has been previously calibrated to zero.

Gyroscope

The sensor estimates the value of delta_theta in a 3D space computing

delta_theta = omega * delta_t , 

where omega must not be intended as the real delta_theta / delta_t, since offset occurs:

omega = omega_real + B + mu , 

where B stands for drift, and mu for the standard deviation (gaussian noise, mean=0). In the analysis, I will consider both. 

In the world of sensor (digital system, in general), formulas as the one above are used instead of the integration: sensors cannot compute any calculus without a defined delta_t interval. In particular, what I have written above is an example of Forward Euler Method).

The weight ("alpha", in the equation in the code) assigned to raw data is proportional to the level of accuracy expected from each sensor, considering both noise and drift (see above): for short time interval "tau", the algorithm gives more value to the measurement from the gyroscope, since the accelerometer may detect high spikes in the change of velocity over time, due to the noise.
The filter, though, still cannot solve offset issue over long intervals: the following example is meant to show how the signal tends to drift, even under the effect of the filter (only by applying Kalman filtering technology can the influence of this random interference be greatly reduced).
Consider alpha = 0.99, and theta_tau = 30° (estimated, not real). At each cycle, the previous value (theta_(tau-1)) is multiplied by 0.99. 
1)	0.99(30)
2)	0.99(0.99*30)
3)	0.99(0.99*0.99*30)
4)	…
   
Within n cycle, theta_tau will be multiplied by 0.99^n (in the example: 30*0.99^3 = 29.1089)
Technically, the filter “low passes” the accelerometer and “high passes” the gyroscope.
A visualization of filter accuracy, at given drift and standard deviation for the gyroscope and at randomly generated Gaussian noise, is in the graph below. 


The Kalman Filter

The Kalman Filter is the core of the simulation proposed in this project. It mathematically weights forecast stability and up-to-date system acquirements, computing the "State Update Equation".

As an example, consider a radar trying to track a plane.

Predicted State: The system's mathematical estimation of its current state, computed from the previous step before checking the actual sensor. It can be: the aircraft is traveling at constant speed.

Measurement Variance: The level of uncertainty or noise inherent to the sensor's readings. A higher variance means the sensor's signal is heavily disturbed and less reliable. It is a value which depends on the sensor's reliability / sensitivity.  

Predicted State Variance: The estimated uncertainty of our mathematical prediction at the current time step. A higher variance means our theoretical model is currently less confident.   

Kalman Gain: The dynamic balancing factor (a scalar in 1D, or a matrix in multidimensional cases) that determines how much the new sensor information should change or correct the predicted state. In the most simplified case (when "u_n" is not considered, see 1)), G is a one-column cinematic matrix which contain the expression for acceleration in function of time. G is implemented and calculated when one between the following situations occurs (most common; in 1) the actual matrix is more complex than a single column, since more and more variable must be considered).

 - 1)If radar has access to the aircraft controls, values such as acceleration are certain and reliable (known external inputs to the system ): in the equation, G is multiplied to "u_n" represents "control variable"
           
 - 2)Else, variable like unpredicted acceleration or wind are treated as process noise: G is multiplied to its transpose matrix (G^T), and the covariance matrix (Q) is obtained.
           
Now, how does it "weight" the two uncertainties (from matrix P, which expresses the doubt of the mathematical model, and R, about system interferences, noise)? It relies on the one with lower covariance.

State Update Equation: The definitive formula that produces the most accurate estimation. It updates the predicted state by applying a correction term, effectively blending the stability of the physical mathematical model with the up-to-date but noisy data from the sensor.   


1D situation

G is a scalar value. For instance, imagine a thermometer. Its noise exists as a sensor noise.   

2D or 3D situation

The Kalman gain is given by a matrix.

Note: in more complex systems, where measurement and the state system belong to different physical domains,  the forecast has to be projected in the measurement domain. It is made possible multiplying the predicted state to the H matrix, before subtracting to the actual measurement (z) (if the ratio is 1:1, H is an identity matrix).  

<img width="1920" height="1080" alt="Screenshot (218)" src="https://github.com/user-attachments/assets/6a24ec72-6dea-464a-995d-80533e654571" />



NOTES TO THE IMAGE

- the closer to 0 the value of "complementary filter accuracy" is, the more accurate is the correction

- in the graph "complementary angle", it is quite evident the offset, which can be more effectively visualized at higher value of "t"

- the initial error spike is caused by the filter initialization dynamics, as weighted data integration begins at the second step ("tau" interval)


___________________________________________________________________________________________________________________________________________________________________________________

Among everyday applications, a notable one is in inertial navigation systems. The random drift rate of the gyroscope will seriously affect the positioning accuracy of the navigation system. Therefore, the inertial navigation system has strict requirements for the random drift rate of the gyroscope, and generally should reach 0.01°/h or even smaller.
Through Kalman esteem, the vehicle can still navigate and track his path precisely, even in absence of GNSS (INS (Inertial Navigation System) or Dead Reckoning).

 
