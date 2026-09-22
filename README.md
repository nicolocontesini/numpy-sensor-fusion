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

Come mostrato nel documento kalman, il processo per un radar si articola così:   
kalman

Stato Predetto (Predicted State):  
x
^
  
n+1,n
​
 =F× 
x
^
  
n,n
​
 +Gu 
n
​
 .   
kalman

L'Equazione di Aggiornamento dello Stato (State Update Equation):
x
^
  
1,1
​
 = 
x
^
  
1,0
​
 +K 
1
​
 (z 
1
​
 − 
x
^
  
1,0
​
 ).   
kalman

Il cuore di questa equazione è il Guadagno di Kalman (K 
1
​
 ). Nel caso 1D, il Guadagno di Kalman è calcolato come:

K 
n
​
 = 
p 
n,n−1
​
 +r 
n
​
 
p 
n,n−1
​
 
​
 

   
DOCX

Dove:

p 
n,n−1
​
  è la varianza dello stato predetto (quanto ci fidiamo del modello teorico).   
DOCX

r 
n
​
  è la varianza della misurazione (quanto ci fidiamo del sensore).   
DOCX


Case: 1D

G is a scalar value. For instance, imagine a thermometer. Its noise exists as a sensor noise.   

Case 2/3D

The Kalman gain is given by a matrix.

Note: in more complex systems, where measurement and the state system belong to different physical domains,  the forecast has to be projected in the measurement domain. It is made possible multiplying the predicted state to the H matrix, before subtracting to the actual measurement (z) (if the ratio is 1:1, H is an identity matrix).   
