# MPC for CHP system
This system represents a simplified combustion-engine combined heat and power (CHP) plant, where waste heat from combustion is transferred to the district heating system via engine boiler heat exchangers. A radiator is employed to extract heat during specific operating conditions, such as engine cold starts.

The outgoing district heating water temperature is maintained within the range $130 \pm 5 ^\circ \mathrm{C}$ under all operating conditions. For instance, during a cold start when components are cold, cooling water must be recirculated to reheat the system. During this initial recirculation phase, the radiator extracts excess heat to prevent the system from overheating.

To design a near-optimal control strategy, we employ a Model-Predictive Control (MPC) approach. In this setup, the radiator power is controlled using the signal $u_\mathrm{R} \in [0, 1]$, while a 3-way valve manages the water recirculation. Deciding when to stop recirculating water is a discrete decision; hence, the valve input $u_\mathrm{V}(t) \in \{0, 1\}$ is treated as a binary decision variable. This configuration results in a mixed-integer MPC problem.

## System model

![image](images/cooling_system.drawio.svg)

This section outlines a simplified first-principles model of the CHP cooling system. The radiator dynamics have been neglected, and heat storage in the components has been significantly simplified to create a more manageable model suitable for demonstrating the MPC control strategy.

### Radiator
The radiator extracts heat form the system and is assumed to be controlled with signal $`u_\mathrm{R}(t)\in[0,1]`$ such that $`\dot{Q}_R = Q_{\mathrm{R,max}}u_R(t)`$ and the dynamics is given by

$`
c_\mathrm{p}M_\mathrm{R}\dot{T}_\mathrm{R} = c_\mathrm{p}\dot{m}T - c_\mathrm{p}\dot{m}T_\mathrm{R} - \dot{Q}_\mathrm{R},
`$

where $M_\mathrm{R}$ represents the internal mass to be heated in the radiator.

### Engine
The engine is a source of heat to the system. The provided heat $\dot{Q}_\mathrm{E}$ is considered to be constant and known. The dynamics of the engine component is

$`
c_\mathrm{p}M_\mathrm{E}\dot{T}_\mathrm{E} = c_\mathrm{p}\dot{m}T_\mathrm{R} - c_\mathrm{p}\dot{m}T_\mathrm{E} + \dot{Q}_\mathrm{E},
`$
where $M_\mathrm{E}$ represents the internal mass to be heated in the engine.

### Boiler
The exhaust-gas boiler is a heat source that is modeled as

$`
c_\mathrm{p}M_\mathrm{B}\dot{T}_\mathrm{B} = c_\mathrm{p}\dot{m}T_\mathrm{E} - c_\mathrm{p}\dot{m}T_\mathrm{B} + \dot{Q}_\mathrm{B},
`$

where $M_\mathrm{B}$ represents the internal mass to be heated in the boiler.

### 3-way valve
The valve simply redistributes flow between the recirculation and outlet pipes. Internal dynamics of the valve has been omitted. The valve position is denoted $u_V(t)\in\{0, 1\}$ and is assumed to be either fully open or closed. The simple valve model results in the algebraic expressions

$`
\begin{align*}
\dot{m}_\mathrm{out} &= \dot{m}u(t), \\
\dot{m}_\mathrm{R} &= \dot{m}(1-u(t)),
\end{align*}
`$

or, equivalently

$`
\dot{m}_{\mathrm{out}} =
\begin{cases}
    \dot{m}, & \text{if }~u_\mathrm{V}(t) = 1, \\
    0, & \text{otherwise,}
\end{cases}
`$

likewise,

$`
\dot{m}_{\mathrm{R}} =
\begin{cases}
    0, & \text{if }~u_\mathrm{V}(t) = 1, \\
    \dot{m}, & \text{otherwise.}
\end{cases}
`$

### Mixing node
The mixing node mixes the recirculated flow and the inlet flows, and is modeled as

$`
\begin{align*}
\dot{m} &= \dot{m}_\mathrm{R} + \dot{m}_\mathrm{in} \\
M_\mathrm{MIX}\dot{T} &= \dot{m}_\mathrm{R}T_\mathrm{R} + \dot{m}_\mathrm{in}T_\mathrm{in} - \dot{m}T.
\end{align*}
`$

Note that, since $`\dot{m}_\mathrm{in} = \dot{m}_\mathrm{out}`$ (conservation of mass), this is equivalent to

$`
\begin{align*}
M_\mathrm{MIX}\dot{T} &=
\begin{cases}
    \dot{m}_\mathrm{in}T_\mathrm{in} - \dot{m}T, & \text{if }~u_\mathrm{V}(t) = 1, \\
    \dot{m}_\mathrm{R}T_\mathrm{R} - \dot{m}T, & \text{otherwise,}
\end{cases} \\
&= 
\begin{cases}
    \dot{m}T_\mathrm{in} - \dot{m}T, & \text{if }~u_\mathrm{V}(t) = 1, \\
    \dot{m}T_\mathrm{R} - \dot{m}T, & \text{otherwise.}
\end{cases} 
\end{align*}
`$

### Combined state-space model
Due to the switching behaviour of the mixing node (caused by the valve), the system dynamics changes depending on if the valve is open or closed. This results in a switching state-space system

$`
\dot{x} =
\begin{cases}
    A_1x + Bu + Fd, & \text{if }~u_\mathrm{V} = 1, \\
    A_0x + Bu + Fd, & \text{otherwise.}
\end{cases} 
`$

where the input, disturbance, and state vectors are

$`
u =
\begin{bmatrix}
u_\mathrm{V}\\
u_\mathrm{R}
\end{bmatrix},
~d =
\begin{bmatrix}
T_\mathrm{in}\\
\dot{Q}_\mathrm{E}\\
\dot{Q}_\mathrm{B}
\end{bmatrix},
~x =
\begin{bmatrix}
T_\mathrm{in}\\
T\\
T_\mathrm{R}\\
T_\mathrm{E}\\
T_\mathrm{B}
\end{bmatrix},
`$

respectively. The system matrices are

$`
\begin{align*}
A_1 &=
\begin{bmatrix}
0 &0 &0 &0 &0\\
\dot{m}/M_\mathrm{MIX} &-\dot{m}/M_\mathrm{MIX} &0 &0 &0\\
0 &\dot{m}/M_\mathrm{R} &-\dot{m}/M_\mathrm{R} &0 &0\\
0 &0 &\dot{m}/M_\mathrm{E} &-\dot{m}/M_\mathrm{E} &0\\
0 &0 &0 &\dot{m}/M_\mathrm{B} &-\dot{m}/M_\mathrm{B}
\end{bmatrix}, \\
A_0 &=
\begin{bmatrix}
0 &0 &0 &0 &0\\
0 &-\dot{m}/M_\mathrm{MIX} &0 &0 &\dot{m}/M_\mathrm{MIX}\\
0 &\dot{m}/M_\mathrm{R} &-\dot{m}/M_\mathrm{R} &0 &0\\
0 &0 &\dot{m}/M_\mathrm{E} &-\dot{m}/M_\mathrm{E} &0\\
0 &0 &0 &\dot{m}/M_\mathrm{B} &-\dot{m}/M_\mathrm{B}
\end{bmatrix},
\\
B &=
\begin{bmatrix}
0 &0\\
0 &0\\
0 &\dot{Q}_\mathrm{R, max}/(c_pM_\mathrm{MAX})\\
0 &0\\
0 &0\\
\end{bmatrix}, \\
F &=
\begin{bmatrix}
1 &0 &0\\
0 &0 &0\\
0 &0 &0\\
0 &1 &0\\
0 &0 &1\\
\end{bmatrix}.
\end{align*}
`$

Furthermore, note that the effect of the recirculation valve $u_V$ only switches the dynamics of the system. Thus the model can be further simplified as

$`
\dot{x} =
\begin{cases}
    A_1x + B_\mathrm{R}u_R + Fd, & \text{if }~u_\mathrm{V} = 1, \\
    A_0x + B_\mathrm{R}u_R + Fd, & \text{otherwise,}
\end{cases} 
`$

where $B_\mathrm{R}$ is the reduced B-matrix.

$`
B_\mathrm{R} =
\begin{bmatrix}
0\\
0\\
\dot{Q}_\mathrm{R, max}/(c_pM_\mathrm{MAX})\\
0\\
0\\
\end{bmatrix}.
`$

### Discrete-time state space
A discrete-time state-space model can be obtained, for example, by zero-order hold ([ZOH](https://en.wikipedia.org/wiki/Discretization#Derivation)) sampling, resulting in the discrete-time switching state-space system $(A_{i,\mathrm{d}}, B_{\mathrm{d}}, F_{\mathrm{d}})$

$`
x(k+1) =
\begin{cases}
    A_{\mathrm{d},1}x(k) + B_{\mathrm{d},1}u_\mathrm{R}(k) + F_{\mathrm{d},1}d(k), & \text{if }~u_\mathrm{V}(k) = 1, \\
    A_{\mathrm{d},0}x(k) + B_{\mathrm{d},0}u_\mathrm{R}(k) + F_{\mathrm{d},0}d(k), & \text{otherwise.}
\end{cases}
`$

with

$$
A_{\mathrm{d},i} = \mathrm{e}^{A_i\Delta T},~B_{\mathrm{d},i} = A_{i}^{-1}(A_{i,\mathrm{d}} - I)B.
$$

> :warning: **NOTE** *Verify this!*   

## Optimization problem
Ideally we would formulate the MPC optimization something like the following:

$`
\begin{align*}
&\underset{x\in\mathbb{R}^{N},e\in\mathbb{R}^{N},u_\mathrm{R}(k)\in\mathbb{R},~u_\mathrm{V}(k)\in \{0, 1\}}{\text{minimize}} \quad 
\sum_{k=0}^{N-1} e(k)^TQ_{e}^{-1}e(k) + \sum_{k=0}^{N-1} u_\mathrm{R}(k)^T Q_{u_\mathrm{R}}^{-1} u_\mathrm{R}(k)\\
&\text{subject to} \\
&x(k+1) = 
\begin{cases}
    A_{\mathrm{d},1}x(k) + B_{\mathrm{d}}u(k) + F_{\mathrm{d}}d(k), & \text{if }~u_\mathrm{V}(k) = 1, \\
    A_{\mathrm{d},0}x(k) + B_{\mathrm{d}}u(k) + F_{\mathrm{d}}d(k), & \text{otherwise.}
\end{cases}, &\quad k=0,1,...,N-1 \\
&e(k) = r(k) - x(k), &\quad k=0,1,...,N-1\\
& x(k) \in [l, u], &\quad k=0,1,...,N-1\\
\end{align*}
`$

However, conventional solvers typically don't accept conditional statements as constraints. Thus, the conditional constraint is rewritten using big-M reformulation as

$`
\begin{align*}
&\underset{x,x_0,x_1\in\mathbb{R}^{N},e\in\mathbb{R}^{N},u_\mathrm{R}(k)\in\mathbb{R},~u_\mathrm{V}(k)\in \{0, 1\}}{\text{minimize}} \quad 
\sum_{k=0}^{N-1} e(k)^TQ_{e}^{-1}e(k) + \sum_{k=0}^{N-1} u_\mathrm{R}(k)^T Q_{u_\mathrm{R}}^{-1} u_\mathrm{R}(k)\\
&\text{subject to} \\
&x_0(k+1) = A_{\mathrm{d},0}x(k) + B_{\mathrm{d}}u(k) + F_{\mathrm{d}}d(k), &\quad k=0,1,...,N-1\\
&x_1(k+1) = A_{\mathrm{d},1}x(k) + B_{\mathrm{d}}u(k) + F_{\mathrm{d}}d(k), &\quad k=0,1,...,N-1\\
&x_1(k+1) - x(k+1) \leq M(1 - u_\mathrm{V}(k)), &\quad k=0,1,...,N-1\\
&x(k+1) - x_1(k+1) \leq M(1 - u_\mathrm{V}(k)), &\quad k=0,1,...,N-1\\
&x_0(k+1) - x(k+1) \leq Mu_\mathrm{v}(k), &\quad k=0,1,...,N-1\\
&x(k+1) - x_0(k+1) \leq Mu_\mathrm{v}(k), &\quad k=0,1,...,N-1\\
&e(k) = r(k) - x(k), &\quad k=0,1,...,N-1\\
& x(k) \in [l, u], &\quad k=0,1,...,N-1\\
\end{align*}
`$

To verify that the two formulations are equivalent, note that, for big-enough $M>0$, if $u_\mathrm{V}(k) = 1$, we have

$`
\begin{align*}
&x_1(k+1) \leq x(k+1) \leq x_1(k) \implies x(k+1) = x_1(k+1), \\
&x_0(k+1) - M \leq x(k+1) \leq x_0(k+1) + M \implies \text{not active contraint}
\end{align*}
`$

Likewise, if $u_\mathrm{V}(k) = 0$,

$`
\begin{align*}
&x_1(k+1) - M \leq x(k+1) \leq x_1(k) + M \implies \text{not active contraint}, \\
&x_0(k+1) \leq x(k+1) \leq x_0(k) \implies x(k+1) = x_0(k+1). \\
\end{align*}
`$
