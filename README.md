# Source Panel Method Validation Using a Van de Vooren Airfoil

## Project Motivation
Computational aerodynamics relies heavily on numerical methods whose accuracy depends on discretization, boundary enforcement, and numerical stability. While high-fidelity CFD tools are widely available, their results are only as reliable as the underlying numerical formulation.

The motivation behind this project was to build and validate a low-order aerodynamic solver from first principles, using a geometry for which an exact analytical solution exists. The Van de Vooren airfoil provides such a case: its geometry and pressure distribution can be derived analytically using conformal mapping, making it an ideal benchmark for validating numerical potential-flow methods.

## Analytical Reference Solution: Van de Vooren Airfoil
The first part of the program generates the airfoil geometry and pressure distribution using the Van de Vooren conformal mapping. 

The code maps a circle in the complex plane to a physical airfoil by applying closed-form transformations. This produces a smooth airfoil with a sharp trailing edge and a prescribed thickness distribution. The geometry is nondimensionalized so the chord extends from $x = -1$ to $x = +1$.

<p align="center">
  <img src="https://i.imgur.com/SdCSkke.png" height="80%" width="80%" alt="Van de Vooren Airfoil Geometry"/>
  <br />
  <i>Figure 2-3: Analytically generated 18% Van de Vooren airfoil used as the numerical benchmark.</i>
</p>

Once the surface coordinates are known, the program evaluates the exact surface velocity field using analytical potential-flow expressions. From these velocities, the pressure coefficient is computed directly using Bernoulli’s equation.

<p align="center">
  <img src="https://i.imgur.com/MG6vbJo.png" height="80%" width="80%" alt="Exact Pressure Distribution"/>
  <br />
  <i>Figure 2-6: Exact pressure coefficient distribution for the 18% airfoil at zero angle of attack.</i>
</p>

This analytical solution is treated as the "ground truth" for the remainder of the project.

## Numerical Method Selection and Rationale
The numerical approach chosen for this study is the **two-dimensional constant-strength source panel method**. This method was selected because:
<br />

* It enforces flow tangency directly on the body surface.
* It is computationally inexpensive.
* Its accuracy is strongly tied to discretization, making convergence studies meaningful.
* It is widely used as a foundation for more advanced panel methods.

## Geometry Discretization and Panel Construction
The analytically generated airfoil is converted into a panel representation by resampling the surface into straight segments of equal arc length. The code interpolates the exact geometry to generate $N+1$ boundary points for a given number of panels $N$.

Discretizations were evaluated for: $N = 10, 20, 30, 50, 100, 250$.

<p align="center">
  <img src="https://i.imgur.com/yOtV5CO.png" height="80%" width="80%" alt="Panel Discretization N=10"/>
  <br />
  <i>Figure 2-10: Panel discretization for N = 10, highlighting geometric approximation at low panel counts.</i>
</p>

## Influence Coefficient Formulation and Boundary Enforcement
At each control point, the no-penetration boundary condition is enforced. The code computes the velocity induced at each control point by every panel using analytical source-panel influence expressions, assembled into a linear system:

$$\sum_{j=1}^{N} C_{ij} q_j = U_\infty \sin(\theta_i - \alpha)$$

Where $q_j$ represents the source strengths on each panel. Self-panel singularities are handled analytically to maintain numerical stability.

## Surface Velocity and Pressure Recovery
Once the source strengths are known, the program computes the tangential velocity ($V_t$) at each control point. The numerical pressure coefficient is then evaluated:

$$C_p = 1 - \left(\frac{V_t}{U_\infty}\right)^2$$

## Numerical Accuracy and Convergence Assessment
To evaluate solver accuracy, the numerical pressure coefficient is plotted against the analytical solution for each panel count.

At low panel counts ($N=10$), discrepancies are most pronounced near the leading edge, where curvature and velocity gradients are highest.

<p align="center">
  <img src="https://i.imgur.com/u9UJgil.png" height="80%" width="80%" alt="Numerical Cp N=10"/>
  <br />
  <i>Figure 2-16: Numerical Cp distribution for N = 10; under-resolution of the leading-edge suction peak is evident.</i>
</p>

As panel resolution increases, the numerical solution converges smoothly toward the analytical result across the entire chord.

<p align="center">
  <img src="https://i.imgur.com/xqrC0Dh.png" height="80%" width="80%" alt="Numerical Cp N=250"/>
  <br />
  <i>Figure 2-21: Near-complete convergence at N = 250, validating the numerical formulation.</i>
</p>

## Engineering Takeaways
<br />

* **Discretization Sensitivity:** Numerical error is dominated by geometric resolution; a low-order source panel method accurately reproduces analytical data given sufficient density.
* **Critical Regions:** Leading-edge regions require significantly higher panel density for accurate pressure recovery due to high curvature.
* **Verification Logic:** This project reflects an emphasis on verification over "black-box" computation, ensuring the solver behaves predictably as discretization is refined.
