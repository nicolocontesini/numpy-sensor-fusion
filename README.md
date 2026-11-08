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

The following section is about the esteem and the comprehension of the filter accuracy, obtained through a mathematical abstraction and analysis. 
My analysis is based both on the study of variance and graphs for different scenarios, each of them characterized by set weight (alpha), noise (mu/ omega_measured) and drift (B/ B_max).
So as to provide a simple and clear representation of the filter mechanism, I wrote a program which simulates it

-	Just like the project objective, this simulation is built upon data acquired from a gyro and an accelerometer
  
-	Np.random.normal function draws random sample values normally distributed around a certain mean value. To get the most realistic value possible, according to the sensitivity toward vibrations (causes of random spikes in the measurement), I set higher spread for the accelerometer (np.random.normal(3.0, 1)). The specific value and spread (3.0 and 1) means that the moving system actually does not experience any change in heading (3.0 degrees). 
-	Consequently, the angular speed (degree/second) mean has been set to 0.0.
  
-	For clarity and simplicity, a “complementary angle” value is set to 3.00 degree: it corresponds to the first one obtained.
  
-	The program computes these values inside the filter equation, and outputs the resulting angle.

As a first take, make reference to the markdown table below

(not to be intended as a test report)

| Tau interval | Accelerometer angle (degree) | Gyro omega (degree/s) | Filtered angle |
| :--- | :--- | :--- | :--- |
| 1 | 2.12863 | -0.65616 | 2.97614 |
| 2 | 2.15961 | -1.06365 | 2.94938 |
| 3 | 2.31481 | -0.47217 | 2.93207 |
| 4 | 2.61712 | -0.40057 | 2.92184 |
| 5 | 4.28818 (SPIKE!!) | 0.90776 | 2.95806 |
  
The following report shows the results obtained at different set values (I focused on alpha and drift). 
Note: alpha, representing the weight assigned to the two sensors, is usually a value that fluctuates between 0.96 and 0.99, causing the “complementary one” going from 0.04 and 0.01 (1-0.96 = 0.04; 1-0.97 = 0.03).

Filter Accuracy Chart
A graphic oversight for the filter accuracy over time is shown by the results chart: for each "tau", the program computes the difference between the filtered value and the real one (true_angle). 

Variance 
So as to get a visible report of different results obtained through different B_max and k-drift, I decided to make the program compute the variance for each set of 100 values, where each value correspond to one measurement acquirement from the sensor. 
The mean value corresponds to true_angle.






NOTES TO THE IMAGE "graph.png" in the repository

- the closer to 0 the value of "complementary filter accuracy" is, the more accurate is the correction

- in the graph "complementary angle", it is quite evident the offset, which can be more effectively visualized at higher value of "t"

- the initial error spike in "complementary accuracy" is caused by the filter initialization dynamics, as weighted data integration begins at the second step ("tau" interval)




___________________________________________________________________________________________________________________________________________________________________________________

Among everyday applications, a notable one is in inertial navigation systems. The random drift rate of the gyroscope will seriously affect the positioning accuracy of the navigation system. Therefore, the inertial navigation system has strict requirements for the random drift rate of the gyroscope, and generally should reach 0.01°/h or even smaller.
Through Kalman esteem, the vehicle can still navigate and track his path precisely, even in absence of GNSS (INS (Inertial Navigation System) or Dead Reckoning).

 
