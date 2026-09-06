# Projectile Motion with Quadratic Drag

A computational physics project using numerical simulation to study projectile motion with and without air resistance. The project starts with ideal projectile motion to test Euler's Method against the analytical solution, then adds quadratic drag to study how air resistance changes the trajectory and optimal launch angle.

The main focus is on scaling. By combining the physical parameters into the dimensionless quantity

$$
\alpha=\frac{cv_0^2}{mg},
$$

the project tests whether different projectiles can have the same normalized behavior even when their individual physical parameters are different.

![Trajectory scaling](images/trajectory_scaling.png)

**Figure 5.** *Normalized trajectories for three values of* $\alpha$, *with each value generated using three different combinations of* $c$, $v_0$, and $m$. *The close overlap shows the scaling predicted by* $\alpha=cv_0^2/(mg)$.

## Table of Contents

* [Motivation](#motivation)
* [Mathematical Model](#mathematical-model)

  * [Ideal Projectile Motion](#ideal-projectile-motion)
  * [Quadratic Air Resistance](#quadratic-air-resistance)
  * [Nondimensionalization](#nondimensionalization)
* [Numerical Method](#numerical-method)
* [Results](#results)

  * [1. Numerical Convergence](#1-numerical-convergence)
  * [2. Numerical Error](#2-numerical-error)
  * [3. Effect of Quadratic Drag](#3-effect-of-quadratic-drag)
  * [4. Optimal Launch Angle and Parameter Scaling](#4-optimal-launch-angle-and-parameter-scaling)
  * [5. Trajectory Scaling Test](#5-trajectory-scaling-test)
* [Key Findings](#key-findings)
* [Future Improvements](#future-improvements)
* [Conclusion](#conclusion)
* [Project Structure](#project-structure)
* [Requirements](#requirements)
* [Running the Project](#running-the-project)

## Motivation

Projectile motion is a useful system for studying numerical methods because the ideal case has an analytical solution that can be used to check the numerical results.

I start with ideal projectile motion to test Euler's Method and see how its accuracy changes with timestep. I then add quadratic air resistance, which makes the equations nonlinear and removes the simple analytical solution.

The main question is whether the behavior of the system can be described using one dimensionless parameter rather than the individual values of the drag coefficient, initial velocity, and mass. Dimensional analysis gives

$$
\alpha=\frac{cv_0^2}{mg}.
$$

This can be interpreted as a measure of the initial drag force compared with the projectile's weight.

The goal is to test whether systems with different values of $c$, $v_0$, and $m$ behave the same way after being scaled if they have the same value of $\alpha$.

## Mathematical Model

The projectile is represented by the state vector

$$
\mathbf{s} =
\begin{bmatrix}
x \\
y \\
v_x \\
v_y
\end{bmatrix}.
$$

The equations of motion are written as a system of first-order differential equations so they can be integrated numerically using Euler's Method.

### Ideal Projectile Motion

For ideal projectile motion,

$$
\frac{dx}{dt}=v_x
$$

$$
\frac{dy}{dt}=v_y
$$

$$
\frac{dv_x}{dt}=0
$$

$$
\frac{dv_y}{dt}=-g.
$$

For an initial speed $v_0$ and launch angle $\theta$, the initial velocity components are

$$
v_{x,0}=v_0\cos\theta,
\qquad
v_{y,0}=v_0\sin\theta.
$$

The analytical solution is

$$
x(t)=x_0+v_0\cos\theta\,t
$$

and

$$
y(t)=y_0+v_0\sin\theta\,t-\frac{1}{2}gt^2.
$$

For a projectile launched and landing at the same height, with $x_0=y_0=0$, the range is

$$
R=\frac{v_0^2\sin(2\theta)}{g}.
$$

This solution gives a reference for checking the numerical Euler solution and measuring its error as the timestep changes.

### Quadratic Air Resistance

The drag force is modeled as

$$
\mathbf{F}_d=-\frac{1}{2}C_d\rho A|\mathbf{v}|\mathbf{v},
$$

where $C_d$ is the drag coefficient, $\rho$ is the air density, and $A$ is the cross-sectional area.

For convenience, these quantities are combined into

$$
c=\frac{1}{2}C_d\rho A.
$$

The resulting accelerations are

$$
a_x=-\frac{c}{m}|\mathbf{v}|v_x
$$

and

$$
a_y=-g-\frac{c}{m}|\mathbf{v}|v_y.
$$

Now the acceleration depends on the projectile's instantaneous velocity, making the system nonlinear.

### Nondimensionalization

The importance of $\alpha$ becomes clearer when the equations are nondimensionalized.

For the quadratic-drag model,

$$
m\frac{d\mathbf{v}}{dt}=
-mg\hat{\mathbf{y}}
-c|\mathbf{v}|\mathbf{v}.
$$

The initial speed $v_0$ provides a natural velocity scale, while $g$ provides the acceleration scale. These give characteristic time and length scales

$$
T=\frac{v_0}{g},
\qquad
L=\frac{v_0^2}{g}.
$$

Define

$$
\mathbf{r}=L\mathbf{r}^\*,
\qquad
t=Tt^\*,
\qquad
\mathbf{v}=v_0\mathbf{v}^\*.
$$

Then

$$
\frac{d\mathbf{v}}{dt}=
g\frac{d\mathbf{v}^\*}{dt^\*}.
$$

Substituting these into the equation of motion gives

$$
mg\frac{d\mathbf{v}^\*}{dt^\*}=
-mg\hat{\mathbf{y}}
-cv_0^2|\mathbf{v}^\*|\mathbf{v}^\*.
$$

Dividing by $mg$ gives

$$
\frac{d\mathbf{v}^\*}{dt^\*}=
-\hat{\mathbf{y}}-
\alpha|\mathbf{v}^\*|\mathbf{v}^\*
$$

where

$$
\alpha=\frac{cv_0^2}{mg}.
$$

After this scaling, $c$, $v_0$, and $m$ no longer appear separately. This suggests that systems with the same $\alpha$ should have the same dimensionless trajectory.

## Numerical Method

The equations are solved using Euler's Method:

$$
\mathbf{s}_{n+1}=
\mathbf{s}_n
+
\Delta t
\frac{d\mathbf{s}}{dt}.
$$

Euler's Method uses the derivative at the beginning of each timestep to approximate the solution. Its local truncation error is $O(\Delta t^2)$ and its global error is $O(\Delta t)$.

The ideal projectile gives an analytical solution that can be used to measure this error directly.

Since the simulation only calculates the projectile at discrete times, the final point will usually not land exactly on $y=0$. If the projectile moves from an above-ground point $(x_1,y_1)$ to a point below the ground $(x_2,y_2)$, I estimate the crossing point by assuming the motion between the two points is approximately linear.

The fraction of the timestep needed to reach $y=0$ is

$$
\alpha=\frac{y_1}{y_1-y_2}.
$$

The corresponding horizontal position is

$$
x_{\mathrm{ground}}=
x_1+\alpha(x_2-x_1).
$$

This interpolated point is used as the final point of the trajectory instead of simply using the first point below the ground.

## Results

### 1. Numerical Convergence

The ideal projectile was simulated using several timestep sizes. As the timestep became smaller, the numerical trajectories moved closer to the analytical solution.

![Ideal Projectile Convergence](images/ideal_projectile.png)

**Figure 1.** *Ideal projectile trajectories for different timestep sizes. The numerical solution approaches the analytical trajectory as the timestep decreases.*

### 2. Numerical Error

I then compared the numerical range with the analytical range for different timestep sizes.

The range error decreases approximately linearly with timestep, which agrees with the first-order global accuracy expected from Euler's Method.

![Error vs. Step Size](images/error_vs_step_size.png)

**Figure 2.** *Range error as a function of timestep for ideal projectile motion. The approximately linear relationship shows the first-order convergence of Euler's Method.*

### 3. Effect of Quadratic Drag

After checking the numerical method against the ideal solution, quadratic air resistance was added to the model.

The drag parameter $c$ was varied while keeping the other physical parameters fixed. As $c$ increases, the projectile experiences stronger air resistance, resulting in shorter ranges and noticeably different trajectories.

![Drag Trajectories](images/drag_trajectories.png)

**Figure 3.** *Projectile trajectories for different values of the quadratic drag parameter* $c$, *with the other physical parameters held fixed.*

### 4. Optimal Launch Angle and Parameter Scaling

Next, I looked at how the optimal launch angle changes when quadratic drag is present.

The angle that produces the maximum horizontal range was found computationally for different values of the drag parameter. A coarse-to-fine search was used to find the maximum without having to simulate every possible angle.

The optimal angle changes as the strength of the drag changes. This raises the question of whether the angle depends specifically on $c$, or whether it depends on a combination of the physical parameters.

Using

$$
\alpha=\frac{cv_0^2}{mg},
$$

I varied $c$, $v_0$, and $m$ independently while keeping $\alpha$ fixed.

The resulting curves overlap closely, suggesting that the optimal launch angle is controlled by $\alpha$ rather than by any one of the individual parameters.

![Optimal launch angle vs drag strength](images/optimal_angle_vs_drag.png)

**Figure 4.** *Optimal launch angle as a function of drag strength. Different combinations of* $c$, $v_0$, and $m$ *produce similar results when they give the same value of* $\alpha$.

### 5. Trajectory Scaling Test

The final test asks whether the scaling with $\alpha$ applies to the entire trajectory, rather than just the optimal launch angle.

I used three values,

$$
\alpha=0.2,\quad 0.5,\quad 0.8,
$$

and created three different parameterizations for each one:

* varying $c$ while keeping $v_0$ and $m$ fixed,
* varying $v_0$ while keeping $c$ and $m$ fixed,
* varying $m$ while keeping $c$ and $v_0$ fixed.

The parameters were chosen so that each system had the same $\alpha$. The trajectories were then normalized using

$$
X=\frac{gx}{v_0^2},
\qquad
Y=\frac{gy}{v_0^2},
$$

and compared.

![Trajectory scaling](images/trajectory_scaling.png)

**Figure 5.** *Normalized trajectories for three values of* $\alpha$, *with each value produced using three different combinations of* $c$, $v_0$, and $m$. *The close overlap agrees with the scaling predicted by* $\alpha=cv_0^2/(mg)$.

The trajectories for the different parameterizations overlap closely for each value of $\alpha$. This supports the prediction from the nondimensionalized equations: once the system is scaled, the trajectory depends on $\alpha$ rather than separately on $c$, $v_0$, and $m$.

## Key Findings

* Euler's Method approaches the analytical solution as the timestep decreases and shows first-order global convergence.
* Quadratic drag reduces the projectile's range and changes the shape of its trajectory.
* The optimal launch angle changes when air resistance is included.
* The dimensionless quantity

$$
\alpha=\frac{cv_0^2}{mg}
$$

provides a measure of the strength of drag relative to gravity.

* Different combinations of $c$, $v_0$, and $m$ with the same $\alpha$ produce closely matching normalized trajectories.
* The results support $\alpha$ as the main dimensionless parameter controlling the scaled behavior of the system.

## Future Improvements

* Add higher-order methods such as RK4.
* Compare the accuracy and computational cost of different numerical methods.
* Add wind to the model.
* Investigate non-uniform air density.
* Allow gravity to vary with altitude.
* Explore analytical or approximate solutions for the quadratic-drag model.

## Conclusion

This project started with a basic simulation of projectile motion and developed into an investigation of how air resistance changes the behavior of the system.

The ideal projectile provided a useful test case for Euler's Method because its analytical solution makes it possible to directly measure numerical error. After adding quadratic drag, the system became nonlinear and no longer had a simple closed-form solution.

The most interesting result came from nondimensionalizing the equations. This led to

$$
\alpha=\frac{cv_0^2}{mg},
$$

which combines the drag coefficient, initial velocity, and mass into a single dimensionless parameter.

I tested this by creating physically different systems with the same value of $\alpha$. After normalizing their trajectories, the results closely overlapped. This suggests that $\alpha$ captures the main scaling behavior of projectile motion with quadratic drag.

## Project Structure

```text
Projectile-Motion-with-Quadratic-Drag/
├── main.py
├── physics.py
├── solvers.py
├── analysis.py
├── requirements.txt
└── README.md
```

## Requirements

* Python 3
* NumPy
* Matplotlib

## Running the Project

### 1. Clone the repository

```bash
git clone https://github.com/rohitkb7782/Projectile-Motion-with-Quadratic-Drag.git
cd Projectile-Motion-with-Quadratic-Drag
```

### 2. Install the dependencies

```bash
pip install -r requirements.txt
```

### 3. Select the Projectile Model

Open `main.py` and select the model using the `model` variable.

For ideal projectile motion:

```python
model = "ideal"
```

For projectile motion with quadratic drag:

```python
model = "drag"
```

### 4. Run the simulation

```bash
python main.py
```
