# numpy-sensor-fusion
In this repository I will explore the main issue (noises) linked to signal acquirement and prove on a graph the effect of filters (Kalman and Complementary Filter) on these results. Starting from a sinusoidal wave to which an artificial Gaussian noise is applied, the projects proves how the statistic and predictive approach (Kalman) and the computationally inexpensive blending of sensor data (Complementary Filter) can rebuild a steady signal from a disturbed one.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The "noise" idea

No measurement in practical application is totally aligned with a "clean" model of data acquirement. The term "noise" is used to describe these casual variation.   

Note: in our simulation, a Gaussian noise will be injected into a sinusoidal wave. This kind of disturb is a mathematical model widely-used for simulation and tests:

- it closely mimics natural interferences, such as the noise produced by digital sensors or hardware fluctuations.

- being based on a normal distribution, it is highly manageable to study, analyze, and implement in software.

- its fluctuactions are simply added to the lean signal, amking it easier to implement algorythm able to eliminate it.   

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


Case: 1D

G is a scalar value. For instance, imagine a thermometer. Its noise exists as a sensor noise.   

Case: 2D or 3D

The Kalman gain is given by a matrix.

Note: in more complex systems, where measurement and the state system belong to different physical domains,  the forecast has to be projected in the measurement domain. It is made possible multiplying the predicted state to the H matrix, before subtracting to the actual measurement (z) (if the ratio is 1:1, H is an identity matrix).   
