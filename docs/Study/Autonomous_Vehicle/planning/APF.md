# Artificial Potential Field (APF)

## 1. Overview

APF (Artificial Potential Field) is a reactive local path planner proposed by Khatib in 1986 for real-time obstacle avoidance of manipulators [Khatib, 1986]. Thanks to its simplicity, ease of implementation, and real-time capability, it has become one of the standard frameworks in robotics and autonomous driving, cited over 7,000 times.

### Core Idea

- The goal generates an attractive potential field $U_{att}(x)$.
- Obstacles generate a repulsive potential field $U_{rep}(x)$.
- The negative gradient of the combined potential $U(x) = U_{att}(x) + U_{rep}(x)$ acts as a force on the robot: $F(x) = -\nabla U(x)$.
- At each timestep the robot moves along $F(x)$, ultimately reaching the goal while avoiding obstacles.

Intuitively, if you visualize $U(x)$ as a landscape of *hills and valleys*, the robot rolls down into the goal valley while detouring around obstacle peaks.

## 2. Formulation

### 2.1 Attractive Potential

![Attractive potential field](<APF/image.png>)

The attractive potential toward the goal $x_{goal}$ is defined as:

$$
U_{att}(x) = \frac{1}{2} K_{att} \cdot \| x - x_{goal} \|^2
$$

- $K_{att}$: attractive gain (tuning parameter)
- $\| \cdot \|$: Euclidean norm

This is a quadratic form, identical to the spring potential energy $\frac{1}{2}kx^2$. The $\frac{1}{2}$ factor is conventionally included so the derivative is clean.

The corresponding attractive force is the negative gradient of the potential:

$$
F_{att}(x) = -\nabla U_{att}(x) = K_{att}(x_{goal} - x)
$$

In 2D, $\nabla$ denotes the vector of partial derivatives $(\partial U / \partial x, \partial U / \partial y)$, pointing in the direction of steepest increase of $U$. Therefore $-\nabla U_{att}$ always points toward the goal, with magnitude proportional to distance.

### 2.2 Repulsive Potential

![Repulsive potential field](<APF/image (1).png>)

The repulsive potential is defined to act only within an influence radius $\rho_0$ of an obstacle:

$$
U_{rep}(x) = \begin{cases} \dfrac{1}{2} \eta \left( \dfrac{1}{\rho(x)} - \dfrac{1}{\rho_0} \right)^2 & \text{if } \rho(x) \leq \rho_0 \\[6pt] 0 & \text{if } \rho(x) > \rho_0 \end{cases}
$$

- $\rho(x) = \| x - x_{obs} \|$: distance from $x$ to the nearest obstacle
- $\rho_0$: influence distance — obstacles farther than this are ignored
- $\eta$: repulsive gain

### Key properties

- $\rho(x) \to 0 \implies U_{rep} \to \infty$ (collision prevention)
- $\rho(x) = \rho_0 \implies U_{rep} = 0$ (continuity at the influence boundary)
- $\rho(x) > \rho_0 \implies U_{rep} = 0$ (distant obstacles are ignored)

The $1/\rho - 1/\rho_0$ design serves two purposes. First, $\rho \to 0$ makes the potential diverge, penalizing collisions infinitely. Second, $\rho = \rho_0$ makes it exactly zero, smoothly stitching the inside and outside of the influence region (no discontinuity).

The corresponding repulsive force is again the negative gradient:

$$
F_{rep}(x) = \eta \left( \frac{1}{\rho} - \frac{1}{\rho_0} \right) \frac{1}{\rho^2} \nabla \rho(x), \quad \text{if } \rho \leq \rho_0
$$

The direction is away from the obstacle ($\nabla \rho$ provides the unit vector).

### 2.3 Total Force and Multiple Obstacles

![Total force with multiple obstacles](<APF/image (2).png>)

The gradient of the combined potential gives the total force on the robot:

$$
F_{total}(x) = F_{att}(x) + \sum_i F_{rep,i}(x)
$$

For multiple obstacles, repulsive forces from each obstacle $i$ are summed as vectors. Obstacles outside $\rho_0$ contribute zero, so the influence-radius cutoff is also a computational efficiency win.

## 3. Path Generation Procedure (Standard Usage)

APF-based path generation works as follows:

```text
while ||F_total|| > epsilon and not goal_reached:
    F = F_att(x) + sum_i F_rep_i(x)
    x <- x + dt * F
    record path point x
```

Termination: $F_{total} \approx 0$ (force equilibrium, ideally at the goal).

## 4. Known Limitations (When Used as a Path Generator)

When APF is used as a path generator, the following intrinsic limitations apply [Koren & Borenstein, 1991]:

### 4.1 Local Minima

At points where the sum of repulsive forces exactly cancels the attractive force, $F_{total} = 0$ and the robot stalls. This is especially common in reversed-C obstacle layouts or symmetric obstacle distributions, and is the principal reason APF cannot guarantee goal reaching.

### 4.2 Goal Non-Reachable with Obstacles Nearby (GNRON)

When the goal lies very close to an obstacle, the repulsive force near the goal stays strong, making the goal unreachable.

### 4.3 Oscillation in Cluttered Environments

In environments with many obstacles, the resultant repulsive vector changes sharply with position, causing oscillations and unstable paths.

### 4.4 Oscillation in Narrow Passages

In a narrow corridor, the repulsive forces from the two walls are nominally balanced, but small disturbances tip the robot toward one wall and the rebound creates a persistent oscillation between walls.

All of the above arise specifically when APF is used as a *gradient-based path generator*.

## 5. Application in This Work: Reframing as Trajectory Evaluation

### 5.1 Motivation

In the hybrid autonomous driving system in this work, the rule-based and E2E planners already produce candidate trajectories. The selector's job is *trajectory evaluation*, not *trajectory generation*. APF is therefore reinterpreted:

- **Standard usage**: convert the gradient of $U(x)$ into a force and generate a path
- **This work**: accumulate the value of $U_{rep}(x)$ along trajectory points to obtain a safety cost

### 5.2 Cost Formulation

For a candidate trajectory $\tau = \{ p_1, p_2, \ldots, p_N \}$, the safety cost is defined as:

$$
J(\tau) = \sum_{p \in \tau} U_{rep}(p) = \sum_{p \in \tau} \frac{1}{2} \eta \left( \frac{1}{\rho(p)} - \frac{1}{\rho_0} \right)^2 \cdot \mathbb{1}[\rho(p) \leq \rho_0]
$$

where $\rho(p)$ is the distance from trajectory point $p$ to the nearest obstacle.

### 5.3 Selection Rule

Between the two candidates $\tau_{RB}$ (rule-based) and $\tau_{E2E}$ (E2E), the selection is:

$$
\tau^* = \arg\min_{\tau \in \{\tau_{RB}, \tau_{E2E}\}} J(\tau), \quad \text{s.t. } \tau \text{ is collision-free}
$$

Collision-free verification is handled separately as a hard filter; $J(\tau)$ is used to compare *safety margins* among collision-free candidates. This is a two-stage decision combining a hard safety constraint with soft proximity scoring.

### 5.4 Why §4 Limitations Don't Apply Here

In this application, none of the §4 limitations apply:

| Limitation                 | In this work | Reason                                      |
| -------------------------- | ------------ | ------------------------------------------- |
| Local minima               | N/A          | No gradient descent is performed            |
| GNRON                      | N/A          | Reaching the goal is not the selector's job |
| Cluttered oscillation      | N/A          | Cost is integrated, not summed as forces    |
| Narrow passage oscillation | N/A          | Same reason                                 |

This is the structural benefit of using APF as a *trajectory evaluation function* rather than a *path generator*.

### 5.5 Why $U_{att}$ Is Not Used

The selector does not use $U_{att}$. Reasons:

- Goal-directed motion is already handled by the candidate planners (rule-based, E2E).
- The selector's responsibility is limited to safety evaluation, in line with the principle of separation of concerns.
- Consequently $K_{att}$ is removed. Furthermore, since $J$ is only used for *relative comparison* between two candidates, $\eta$ does not affect the comparison outcome — the formulation effectively reduces to the single parameter $\rho_0$.

## 6. Parameter Summary

| Symbol      | Meaning             | Used in this work                    | Initial value |
| ----------- | ------------------- | ------------------------------------ | ------------- |
| $K_{att}$   | Attractive gain     | Not used                             | —             |
| $\eta$      | Repulsive gain      | Nominally used (no effect on result) | 1.0           |
| $\rho_0$    | Influence radius    | Key parameter                        | 5.0 m         |

## References

- Khatib, O. (1986). *Real-time obstacle avoidance for manipulators and mobile robots*. The International Journal of Robotics Research.
- Koren, Y., & Borenstein, J. (1991). *Potential field methods and their inherent limitations for mobile robot navigation*. ICRA.
