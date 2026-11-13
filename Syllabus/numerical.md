---
header-includes:
  - \usepackage{longtable}
  - \usepackage{array}
---

# Numerical Methods for Geosciences: Building an Earthquake Cycle Model
### Fridays 10:30–12:30, Sept 2026–Jan 2027 (Salle E509)
### Instructors: Harsha S. Bhat, Xavier Perrot

This course aims at training students in numerical methods in geosciences. While the topic is very vast, in the interest of making it useful and goal-oriented, we have decided to build an earthquake-cycle model by the end of the course. The course is about learning aspects of numerical methods based on the tools needed to build this model — root-finding and time integration schemes to handle the governing nonlinear equations; finite differences and finite volumes to solve the thermal pressurization diffusion problem on the fault; and boundary element methods to compute the elastic interaction between fault patches. These pieces are then coupled together, with dilatancy and adaptive time-stepping added, to assemble a working quasidynamic earthquake-cycle simulator on a 1D fault. Each theoretical lecture is paired with a hands-on coding session in Python, so that by the end of the semester students will have built, from scratch, every component of a functioning numerical model of the earthquake cycle.

Each topic gets a full 2h theory Friday and, the following Friday, a full 2h coding
Friday — a week apart. Every coding session opens with a short recap of the prior week's
theory before students start writing code.

**Session calendar**

\begin{longtable}{|c|c|c|p{10cm}|}
\hline
\textbf{\#} & \textbf{Date} & \textbf{Type} & \textbf{Topic} \\
\hline
\endhead
1  & Fri 11 Sep 2026 & Theory & Foundations --- root-finding \& time integration \\
\hline
2  & Fri 18 Sep 2026 & Coding & Foundations \\
\hline
3  & Fri 09 Oct 2026 & Theory & FD --- thermal pressurization I \\
\hline
4  & Fri 16 Oct 2026 & Coding & FD \\
\hline
5*  & Fri 23 Oct 2026 & Theory & FV --- thermal pressurization II \\
\hline
6*  & Fri 30 Oct 2026 & Coding & FV \\
\hline
7  & Fri 06 Nov 2026 & Theory & BEM --- elastic interaction matrix \\
\hline
8  & Fri 13 Nov 2026 & Coding & BEM \\
\hline
9  & Fri 20 Nov 2026 & Theory & Coupling --- mechanics + TP + dilatancy + adaptive integration \\
\hline
10 & Fri 04 Dec 2026 & Coding & Coupling \\
\hline
11 & Fri 11 Dec 2026 & Theory & Synthesis --- multi-patch assembly \& benchmarking \\
\hline
12 & Fri 18 Dec 2026 & Coding & Synthesis \\
\hline
\end{longtable}
* Vacances de la Toussaint


## Sessions 1–2 — Foundations: root-finding & time integration

**Session 1 (Theory, 2h)**

- Rate-and-state friction law inversion via Newton–Raphson: derivation, convergence, why a
  log-V initial guess behaves better than a raw-V one.
- A proper numpy/vectorization review — array ops, functions, plotting — this pays off
  directly in Sessions 7–8 (assembling and multiplying the stiffness matrix) and Sessions
  9–10 (coupling), where shaky fundamentals would otherwise slow things down.
- Explicit ODE integration: Euler from a Taylor expansion, then RK4.
- The single-DOF spring-slider system: dτ/dt = k(V_load − V), V solved via Newton each step.

**Session 2 (Coding, 2h)**

- Recap (10–15 min): Newton's method and RK4, core ideas only.
- Newton solver + friction-law wrapper.
- RK4 integrator; run the baseline spring-slider, plot V(t).


---

## Sessions 3–4 — FD: thermal pressurization I

**Session 3 (Theory, 2h)**

- Mechanism: frictional heating q = τV at the fault plane drives 1D diffusion of temperature
  and pore pressure perpendicular to the fault; pore-pressure rise reduces effective normal
  stress, weakening the fault (Rice, 2006).
- Full derivation of the FD scheme: discretize the perpendicular (y) direction, central-
  difference the second derivative, explicit (FTCS) vs. implicit (backward Euler, tridiagonal
  solve) time-stepping.
- Stability, derived properly : von Neumann analysis, the diffusion
  number, and why implicit will matter once this couples to a fast-accelerating slip velocity.
- Boundary conditions: source at y = 0, effectively-infinite far-field condition.

**Session 4 (Coding, 2h)**

- Recap: the diffusion equation and the FD scheme.
- Build a 1D FD diffusion solver for T(y,t) given a prescribed heating history q(t) at y=0.
- Validate against the analytic Green's-function solution for a delta-function heat pulse.

---

## Sessions 5–6 — FV: thermal pressurization II

**Session 5 (Theory, 2h)**

- Full control-volume / flux-balance derivation of the same diffusion equation — same result
  as Session 3, different route.
- Why FV handles a spatially varying diffusivity properly (a diffusivity contrast between
  fault-core and damage-zone material) where naive FD needs care.
- Work through the uniform-diffusivity case (should agree with FD) and a diffusivity-jump
  case.

**Session 6 (Coding, 2h)**

- Recap: control-volume derivation, where FD and FV should and shouldn't agree.
- Rewrite the diffusion solver in flux-conservative FV form.
- Run both on a spatially varying diffusivity profile and compare.

---

## Sessions 7–8 — BEM: the elastic interaction matrix

**Session 7 (Theory, 2h)**

- Discretizing the 1D fault into N patches of slip.
- Full derivation of the elastic dislocation kernel for a 1D fault — From first
  principles rather than handing it over as a formula, especially if students haven't taken
  the earthquake source mechanics course.
- Assemble into a stiffness matrix K (N×N): symmetry, the self-stiffness diagonal term, why
  nearby patches dominate the interaction.

**Session 8 (Coding, 2h)**

- Recap: the kernel derivation and matrix structure.
- Validate on a simple two-patch symmetric case.

---

## Sessions 9–10 — Coupling: mechanics, thermal pressurization, dilatancy & adaptive integration

**Session 9 (Theory, 2h)** — the pinch-point lecture; it now gets the room it deserves.

- Full per-patch picture: elastic stress from K (BEM) -> root-find V from the friction law
  using an *effective* normal stress σ_eff = σn − p (not the constant σn from Session 1) ->
  frictional heat q = τV feeds the diffusion solver -> updated p feeds back into σ_eff next
  step.
- Why this feedback loop (heat -> pressure -> weakening -> faster slip -> more heat) is stiffer
  than the Session 1.
- Dilatancy as a second, competing feedback: gouge porosity φ relaxes toward a
  velocity-dependent steady state over the same critical slip distance D_c already used in
  the friction law — structurally the same relaxation-ODE pattern as the state variable θ
  from Session 1, just applied to porosity instead of friction state. Point this out
  explicitly; it's new physics.
- Operator splitting, worked by hand on the board with a small 2-coupled-ODE example before
  touching the real system: advance the mechanical ODE one step with the current p, then
  advance the diffusion solve (now with both the heating source and the dilatant sink) one
  step, alternate.

**Session 10 (Coding, 2h)**

- Recap: the feedback loop, the splitting scheme, and the two competing pressure terms —
  referring back to the board example.
- Wire in a diffusion module FD or FV, student's choice, same 
  interface — behind the friction/root-finding step.
- Add the porosity state variable and its relaxation ODE; feed the dilation
  rate into the diffusion sink term.
- Re-run the Session 1–2 spring-slider three ways: dry baseline, TP only, TP + dilatancy —
  compare all three directly.

---

## Sessions 11–12 — Synthesis: full multi-patch assembly and benchmarking

**Session 11 (Theory, 2h)**

- Extending the Session 9–10 feedback loop to N patches: each patch carries its own
  independent local diffusion state and its own porosity state (a small perpendicular grid
  per patch), coupled to the others only through the elastic stress-transfer matrix — not
  directly coupled in the perpendicular direction. Diffusion solves stay embarrassingly
  parallel: one independent tridiagonal system per patch, per step.
- Diagnostics: moment-rate function, recurrence interval, and a meaningful
  dry / TP-only / TP+dilatancy comparison across the whole fault — this is where dilatancy's
  stabilizing role against thermal weakening actually becomes visible at the cycle scale,
  not just the single-patch scale from Session 10.
- Benchmarking: the single-patch limit should recover the Session 9–10 result; check
  simulated nucleation length against the analytic h* estimate.

**Session 12 (Coding, 2h)** — This is where all six
prior pieces complexify, and bugs are hardest to isolate without dedicated time.

- Recap: multi-patch extension and the diagnostics to compute.
- Assemble: N patches, each with its own diffusion and porosity state,
  coupled via elasticity, driven by the Session 9 RHS.
- Run at moderate N (~50), produce moment-rate/recurrence plots for the dry, TP-only, and
  TP+dilatancy cases side by side.

---