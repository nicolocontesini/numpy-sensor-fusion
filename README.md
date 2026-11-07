# numpy-sensor-fusion
In this repository I will explore the main issue (noises) linked to signal acquirement and prove on a graph the effect of filters (Complementary and Kalman Filter) on these results. Starting from a sinusoidal wave to which an artificial Gaussian noise is applied, the projects proves how the computationally inexpensive blending of sensor data (Complementary Filter) can rebuild a steady signal from a disturbed one. The signal is meant to represent the value of a certain angle theta (in the graph expressed in degree) over a "t" seconds interval. For simplicity, I have chosen a 2D case, where we can imagine a drone flying.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The "noise" idea

No measurement in practical application is totally aligned with a "clean" model of data acquirement. The term "noise" is used to describe these casual variations.   

Note: in our simulation, a Gaussian noise will be injected into a sinusoidal wave. This kind of disturb is a mathematical model widely-used for simulation and tests:

- it closely mimics natural interferences, such as the noise produced by digital sensors or hardware fluctuations.

- since it is based on a normal distribution, it is highly manageable to study, analyze, and implement in software.

- its fluctuations are simply added to the lean signal, making it easier to implement algorithm able to eliminate it.

- the “Central Limit Theorem” states that the sum of a large number of independent random variables (as in this simulation) tends toward a Gaussian bell-curve distribution


The Complementary Filter

A complementary filter is a quick and effective method for blending measurements from an accelerometer and a gyroscope to generate an estimate for orientation.

The filter works assigning specific weights to the measurements provided by the two sensors, so as to:

-	Smooth spikes measured from the accelerometer
  
-	Reduce the offset caused by the gyroscope drift
Before moving on a brief description of the two sensors considered for this simulation, consider the following statement, necessary to understand the conclusion drawn from each experiment.
In the complementary filter, the value of Bias settles at a constant limit, called B_max, described by the following equation:
B_max = B[tau] / 1 – exp(-k_drift * t)
Where k is a constant which expresses how “fast” B(tau) is growing to B_max, t stands for time since the filter started.
Such specification is necessary because, in absence of this formula, the simulation would have lost the crucial detail of the offset: by simply adding B to the to the true angular velocity value, the growing offset would not be visible, since a graph plotting the measured velocity would only represent “true_speed” simply offset of a value B.

Accelerometer

A 3D accelerometer estimates the tilt angles (pitch and roll) by measuring the projections of the gravity vector. For instance, pitch is computed as:
theta = arctan(a_x / sqrt((a_y)^2 + (a_z)^2))
where a represents the acceleration measured along each respective axis (x, y, z). 
However, accelerometers are highly sensitive to high-frequency noise: mechanical vibrations and external forces temporarily add up to the gravity vector, instantly distorting the true angular measurement.
For this analysis, I assume that the sensor is only affected from noises from the system’s vibrations, and any static bias has been previously calibrated to zero.

Gyroscope
A 3D gyroscope is a sensor which measures the angular rate by applying the theory of the Coriolis effect, which refers to the force of inertia that acts on objects in motion in relation to a rotating frame.
It is composed of a mass which, when experiences a force, causes a change in the value of capacitance which is then converted into an electrical signal proportional to the angular rate.
The sensor estimates the value of delta_theta in a 3D space computing
delta_theta = omega * dt ,
where omega must not be intended as the real delta_theta / delta_t, since offset (B) and micro-vibrations (mu) occur:
omega = omega_real + B + mu.
In my code, I decided to consider a constantly generated Gaussian noise to simulate disturbs such as micro-vibrations :
omega_measured = true_speed + B + omega_noise.

The weight

The weight ("alpha", in the equation in the code) assigned to raw data is proportional to the level of accuracy expected from each sensor, considering both noise and offset (see above): for short time interval "tau", the algorithm expects higher reliability to the measurement from the gyroscope, since the accelerometer may detect high spikes in the change of pitch over time, due to the noise.

Technically speaking, the filter “low passes” the accelerometer and “high passes” the gyroscope.
Thanks to these two sources and weights, the filter can counterbalance the tendence to drift of the gyroscope and smooth eventual noise spikes detected from the accelerometer.

The Kalman Filter

In "filter.py", consider:

x = State Vector: the true underlying variables to estimate ; x_minus is the previous

p = Predicted State Variance: a measure of the estimated accuracy or uncertainty of the state estimate ; p_minus is the previous

k_gain = Kalman Gain: "weights" how much the algorithm should rely more on the physical model or the sensor data.

R = Measurement Variance Mtrix

Q = Process Noise Matrix

x = x_minus + k_gain * (accel_angle - x_minus) = State Update Equation


The Kalman Filter is the core of the simulation proposed in this project. It mathematically weights forecast stability and up-to-date system acquirements, computing the "State Update Equation".

NOTE: in the situation analyzed, for simplicity, I assume there is no input (it means that the Filter doesn't include in the computing any input, like plane cockpit command) : G*u_n =0 (see below for complete variable explanation).

As an example, consider a radar trying to track a plane.

Predicted State: The system's mathematical estimation of its current state, computed from the previous step before checking the actual sensor. It can be: the aircraft is traveling at constant speed.

Measurement Variance: The level of uncertainty or noise inherent to the sensor's readings. A higher variance means the sensor's signal is heavily disturbed and less reliable. It is a value which depends on the sensor's reliability / sensitivity.  

Predicted State Variance [p]: the estimated uncertainty of the mathematical prediction at the current time step. A higher variance means the theoretical model is currently less confident.   

Kalman Gain [k_gain]: the dynamic balancing factor (a scalar in 1D, or a matrix in multidimensional cases) that determines how much the new sensor information should change or correct the predicted state. In the most simplified case (when "u_n" is not considered, see 1)), G is a one-column cinematic matrix which contain the expression for acceleration in function of time. G is implemented and calculated when one between the following situations occurs (most common in 1) the actual matrix is more complex than a single column, since more and more variable must be considered).

 - 1)If radar has access to the aircraft controls, values such as acceleration are certain and reliable (known external inputs to the system ): in the equation, G is multiplied "u_n" ("control variable").
           
 - 2)Else, variable like unpredicted acceleration or wind are treated as process noise: [G] is multiplied to its transpose matrix (G^T), and the covariance matrix [Q] is obtained.
           
Now, how does it "weight" the two uncertainties (from matrix [p], which expresses the doubt of the mathematical model, and [R], about system interferences, noise)? It relies on the one with lower covariance.

State Update Equation: The definitive formula that produces the most accurate estimation. It updates the predicted state by applying a correction term, effectively blending the stability of the physical mathematical model with the up-to-date but noisy data from the sensor.   


1D situation

[G] is a scalar value. For instance, imagine a thermometer. Its noise exists as a sensor noise.   

2D or 3D situation

The Kalman gain is given by a matrix.

Note: in more complex systems, where measurement and the state system belong to different physical domains,  the forecast has to be projected in the measurement domain. It is made possible multiplying the predicted state to the H matrix, before subtracting to the actual measurement (z) (else, like in this project, H is an identity matrix).  




NOTES TO THE IMAGE "graph.png" in the repository

- the closer to 0 the value of "complementary filter accuracy" is, the more accurate is the correction

- in the graph "complementary angle", it is quite evident the offset, which can be more effectively visualized at higher value of "t"

- the initial error spike in "complementary accuracy" is caused by the filter initialization dynamics, as weighted data integration begins at the second step ("tau" interval)




___________________________________________________________________________________________________________________________________________________________________________________

Among everyday applications, a notable one is in inertial navigation systems. The random drift rate of the gyroscope will seriously affect the positioning accuracy of the navigation system. Therefore, the inertial navigation system has strict requirements for the random drift rate of the gyroscope, and generally should reach 0.01°/h or even smaller.
Through Kalman esteem, the vehicle can still navigate and track his path precisely, even in absence of GNSS (INS (Inertial Navigation System) or Dead Reckoning).

 
