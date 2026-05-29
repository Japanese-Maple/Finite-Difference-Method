### `Poisson_Equation` (Elliptic Solver)

$$\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} = f(x,y)$$

Implemented in **`solpo.py`**, this class handles steady-state boundary value problems where the system has reached a state of equilibrium under an external driving force $f(x,y)$. It sets up a discrete 2D Laplacian operator matrix using sparse tensor products (`scipy.sparse.kron`) and directly solves the static system across an $(n \times n)$ spatial grid using an optimized direct sparse matrix solver (`spsolve`). The script also features an automated system that reads the raw Python code of the input function and translates it into formal $\LaTeX$ typography for the final rendered multi-panel analysis dashboard.

---

### `Heat_Equation` (Parabolic Solver)

$$\frac{\partial u}{\partial t} = \alpha \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right)$$

Implemented in **`solhe.py`**, this class simulates how thermal energy diffuses through a two-dimensional medium over time under a diffusion coefficient $\alpha$. The class implements a time-dependent explicit finite difference loop that calculates the next state of the system based on its current state, while continuously updating four independent boundary conditions along the outer edges of the grid at every time step. To ensure the simulation remains stable and does not diverge numerically, the time step $\Delta t$ is strictly limited based on the spatial resolution $\Delta x$ to satisfy the mathematical CFL stability constraint.

---

### `Wave_Equation` (Hyperbolic Solver)

$$\frac{\partial^2 u}{\partial t^2} = c^2 \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right)$$

Implemented in **`solwe.py`**, this class models the physical propagation of displacement waves (such as ripples on a surface) through a dynamic spatial domain. It solves a second-order time-stepping scheme using a leap-frog integration style, meaning it references both the current and previous time steps to compute the system's acceleration. A key feature of this class is its capability to enforce zero-boundary Dirichlet conditions across irregular, non-rectangular spaces by applying custom geometric masks (such as a circle or ellipse) directly to the flattened grid at every step of the simulation loop.
