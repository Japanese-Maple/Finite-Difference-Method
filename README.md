### `Poisson_Equation` (Elliptic Solver)

$$\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} = f(x,y)$$

Implemented in **`solpo.py`**, this class handles steady-state boundary value problems where the system has reached a state of equilibrium under an external driving force $f(x,y)$. It sets up a discrete 2D Laplacian operator matrix using sparse tensor products (`scipy.sparse.kron`) and directly solves the static system across an $(n \times n)$ spatial grid using an optimized direct sparse matrix solver (`spsolve`). The script also features an automated system that reads the raw Python code of the input function and translates it into formal $\LaTeX$ typography for the final rendered multi-panel analysis dashboard.

---

### `Heat_Equation` (Parabolic Solver)

$$\frac{\partial u}{\partial t} = \alpha \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right)$$

Implemented in **`solhe.py`**, this class simulates how thermal energy diffuses through a two-dimensional medium over time under a diffusion coefficient $\alpha$. The class implements a time-dependent explicit finite difference loop that calculates the next state of the system based on its current state, while continuously updating four independent boundary conditions along the outer edges of the grid at every time step. To ensure the simulation remains stable and does not diverge numerically, the time step $\Delta t$ is strictly limited based on the spatial resolution $\Delta x$ to satisfy the mathematical CFL stability constraint.

<p align="center">
  <video src="https://raw.githubusercontent.com/Japanese-Maple/Finite-Difference-Method/main/Demonstrations/Outputs/Heat_Equation_Simulation_2D.mp4" width="700" controls autoplay loop muted></video>
</p>

---

### `Wave_Equation` (Hyperbolic Solver)

$$\frac{\partial^2 u}{\partial t^2} = c^2 \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right)$$

Implemented in **`solwe.py`**, this class models the physical propagation of displacement waves (such as ripples on a surface) through a dynamic spatial domain. It solves a second-order time-stepping scheme using a leap-frog integration style, meaning it references both the current and previous time steps to compute the system's acceleration. A key feature of this class is its capability to enforce zero-boundary Dirichlet conditions across irregular, non-rectangular spaces by applying custom geometric masks (such as a circle or ellipse) directly to the flattened grid at every step of the simulation loop.

<p align="center">
  <video src="Demonstrations/Outputs/Wave_Equation_Simulation_2D.mp4" width="700" controls autoplay loop muted></video>
</p

---

### Video Stitching and Asynchronous Parallelization

Generating high-resolution 2D contours and 3D surface frames over long time series creates a severe bottleneck when executed sequentially on a single CPU core. To bypass this, the framework parallelizes frame rendering across an arbitrary pool of worker threads using `joblib.Parallel`.

```
[Time Steps 1...T] ──> [Joblib Parallel Pool] ──> [Asynchronous Worker Cores]
                                                               │
[Saved Video] <── [FFmpeg Libx264 Engine] <── [Target Output Directory (.png)]

```

The rendering environment uses a headless backend (`matplotlib.use('Agg')`) to generate and serialize individual frames concurrently. Once the frame sequence is complete, `ffmpeg` compiles the files directly into a single video asset via native command execution:

* **`-vf "scale={width}:{height}:flags=lanczos"`**: Re-samples dimensions to satisfy strict H.264 even-pixel constraints while applying high-quality Lanczos sinc filtering to eliminate spatial aliasing in dynamic gradients.
* **`-c:v libx264 -crf 18`**: Invokes the H.264 encoder at a Constant Rate Factor of 18, delivering visually lossless output by dynamically adjusting spatial compression ratios relative to frame-to-frame motion deltas.
* **`-pix_fmt yuv420p`**: Transcodes arrays into standard 4:2:0 planar chroma subsampling to guarantee native playback compatibility inside downstream IPython notebook display engines.

---

### High-Performance Sparse Kronecker Algebra

Evaluating a standard five-point central finite difference scheme across a continuous 2D spatial domain using nested scalar loops incurs an inefficient $\mathcal{O}(n^4)$ computational complexity. The framework optimizes this by abstracting the continuous discrete spatial operator $\nabla^2 u$ into a vectorized linear system:

$$L = (I \otimes D) + (B \otimes I)$$

Where $\otimes$ denotes the Kronecker tensor product. This operation replicates the 1D second-derivative stencil matrix $D$ across the identity matrix $I$ of the orthogonal axis, effectively coupling the independent spatial variables.

```
   [1D Finite Difference Stencil]             [2D Sparsity Matrix Structure]
          [ -1   4  -1 ]                [ D  -I   0   0 ]  where D contains
                                        [-I   D  -I   0 ]  the main 1D diagonal
                                        [ 0  -I   D  -I ]  and -I defines the
                                        [ 0   0  -I   D ]  spatial grid coupling

```

Because an $(n \times n)$ spatial grid yields an operator matrix of size $(n^2 \times n^2)$, dense storage formats would rapidly exhaust hardware memory constraints. The framework constructs this matrix exclusively under a Compressed Sparse Row (`CSR`) schema using `scipy.sparse`, tracking only non-zero coordinates (`rows, cols, values`).

This structural optimization dramatically accelerates performance:

1. **Dynamic Evolutionary Systems (Parabolic/Hyperbolic Solvers):** Explicit time-dependent integration steps collapse into optimized sparse matrix-vector products (`L @ v[t-1, :]`), reducing processing time to linear complexity $\mathcal{O}(n^2)$ within cache memory.
2. **Static Equilibrium Systems (Elliptic Solver):** The zero-order system $L u = f$ is passed directly to `scipy.sparse.linalg.spsolve`. It bypasses slow, iterative relaxation methods (e.g., Jacobi or Gauss-Seidel) by performing direct SuperLU sparse LU factorization, computing the complete global solution instantly.
