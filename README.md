# Force Decomposition and Resultant Vector  
We break each force into horizontal (x) and vertical (y) components using standard trigonometry: \(F_x = F\cos\theta\), \(F_y = F\sin\theta\)【22†L294-L300】.  For our beam (with +x to the right, +y up), the forces decompose as:  
- *F₁ (10 N, at A, vertical down)*: \(F_{1x}=0,\ F_{1y}=-10.0\) N.  
- *F₂ (15 N at x=4 cm, 60° below +x)*: \(F_{2x}=15\cos(-60°)=+7.50\) N,  \(F_{2y}=15\sin(-60°)=-12.99\) N.  
- *F₃ (20 N at x=8 cm, 30° below –x)*: \(F_{3x}=20\cos(180°+30°)=-17.32\) N, \(F_{3y}=20\sin(180°+30°)=-10.00\) N.  
- *F₄ (5 N at x=12 cm, vertical down)*: \(F_{4x}=0,\ F_{4y}=-5.00\) N.  

Summing components gives the net (resultant) force:  
\[R_x = 0 + 7.50 - 17.32 + 0 = -9.82\text{ N},\quad R_y = -10.0 -12.99 -10.00 -5.00 = -37.99\text{ N}.\] 

- **Resultant magnitude:** Using the Pythagorean theorem 【19†L245-L247】, \(R = \sqrt{R_x^2 + R_y^2} = \sqrt{(-9.82)^2 + (-37.99)^2} \approx 39.24\) N.  
- **Resultant direction:** The angle θ of R from the +x-axis satisfies \(\tan\theta = R_y/R_x\)【22†L298-L300】.  Numerically, \(\tan\theta = 37.99/9.82 \approx 3.87\), so \(\arctan(3.87)=75.5°\).  Since both \(R_x<0\) and \(R_y<0\), the resultant lies in the third quadrant.  Thus \(\theta \approx 180° + 75.5° = 255.5°\) measured CCW from +x.  Equivalently, the resultant points *15.5° below* the negative x–direction (down-left).  

# Moments and Varignon’s Theorem  
To check equilibrium and find the line of action, we compute the net moment about A (taking counterclockwise as positive).  Varignon’s theorem tells us that the total moment equals the sum of the moments of the components【5†L801-L804】.  In practice we calculate each vertical component times its lever arm (horizontal distance):  
\[M_A = \sum (F_y \times x).\]  
Here, \(F_{1y}\) at A has no lever arm; the others give  
\[M_A = ( -12.99\,\text{N} ) (4\,\text{cm}) + (-10.00\,\text{N})(8\,\text{cm}) + (-5.00\,\text{N})(12\,\text{cm}) = -51.96 - 80.00 - 60.00 = -191.96\text{ N·cm}.\]  
The negative sign indicates a clockwise (CW) moment.  By definition, **Moment = Force×distance**【28†L133-L137】, so this is the same as a single 39.24 N force acting at some perpendicular distance \(d\) producing 191.96 N·cm of torque.  In magnitude, \(d = |M_A|/R = 191.96/39.24 \approx 4.89\) cm.  (The sign convention shows the line-of-action lies on the side of A opposite to the force direction.)  

# Resultant Direction (Angle) Explained  
Summarizing the direction: the resultant force is 39.24 N and points down and to the left (west of south).  The computed angle \(255.5°\) from +x is “below the leftward horizontal.”  This is found systematically by the arctan rule【22†L298-L300】 and adjusting for quadrant.  In engineering terms, this is the *vector sum* of all forces (the parallelogram law).  Practically, one could also draw the component forces head-to-tail to see the same result.  

# Context and Theorems  
This analysis follows fundamental static and vector principles.  For example, Varignon’s theorem【5†L801-L804】 (and the Principle of Moments) is essentially Newton’s first law for rotation: no net torque if moments balance.  Real-world applications abound: engineers always replace complex force systems by a single resultant when designing structures or machines.  For instance, the forces acting on a building beam (wind, weight, etc.) are summed in this way to find the net load and its line of action.  Likewise, tools like a wrench exploit the moment = force×lever-arm rule【28†L133-L137】: applying the same force farther out increases torque.  In all cases, the *direction* of the resultant (found via \(\arctan\) and quadrant logic) tells us exactly where the net push or pull acts.  

**Answer:** The forces sum to \(R_x=-9.82\) N, \(R_y=-37.99\) N, so \(R\approx39.24\) N.  Its direction is \(\theta=\arctan(R_y/R_x)\approx255.5^\circ\) (i.e. down-left).  This follows from standard vector addition【22†L294-L300】【19†L245-L247】.  Using Varignon’s theorem【5†L801-L804】【28†L133-L137】 confirms the net moment about A (≈191.96 N·cm CW) and shows the resultant acts about 4.89 cm from A.  These results match the physics of rigid bodies (no net rotation if support reactions compensate the 39.24 N at that distance).  In short, we have rigorously derived the resultant’s magnitude and angle as above (consistent with standard engineering statics).  
