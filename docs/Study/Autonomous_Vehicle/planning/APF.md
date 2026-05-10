# Artificial Potential Field (APF)

## 1. 개요

APF (Artificial Potential Field)는 1986년 Khatib이 매니퓰레이터의 실시간 obstacle avoidance를 위해 제안한 reactive local path planner이다 [Khatib, 1986]. 알고리즘 자체가 단순하여 구현이 용이하고 실시간 동작이 가능하다는 장점으로 인해 robotics 및 자율주행 분야에서 7,000회 이상 인용된 표준 framework 중 하나이다.

### 핵심 원리

- 목표지점은 인력 potential field $U_{att}(x)$ 를 형성한다.
- 장애물은 척력 potential field $U_{rep}(x)$ 를 형성한다.
- 두 potential의 합 $U(x) = U_{att}(x) + U_{rep}(x)$ 의 negative gradient가 로봇에 작용하는 force가 된다: $F(x) = -\nabla U(x)$.
- 로봇은 매 timestep마다 $F(x)$ 방향으로 이동하며, 결과적으로 장애물을 회피하면서 목표에 도달한다.

직관적으로, $U(x)$ 를 *언덕과 골짜기*로 시각화하면 로봇은 목표 지점의 골짜기로 굴러 떨어지면서 장애물의 봉우리를 우회한다.

## 2. Formulation

### 2.1 Attractive Potential

![Attractive potential field](<APF/image.png>)

목표지점 $x_{goal}$ 에 대한 인력 potential을 다음과 같이 정의한다:

$$
U_{att}(x) = \frac{1}{2} K_{att} \cdot \| x - x_{goal} \|^2
$$

- $K_{att}$: attractive gain (튜닝 파라미터)
- $\| \cdot \|$: Euclidean norm

이는 스프링 위치에너지 $\frac{1}{2}kx^2$ 와 동일한 quadratic 형태이다. 미분 시 깔끔한 결과를 얻기 위해 $\frac{1}{2}$ 계수가 관습적으로 포함된다.

대응되는 attractive force는 potential의 negative gradient로 주어진다:

$$
F_{att}(x) = -\nabla U_{att}(x) = K_{att}(x_{goal} - x)
$$

2D 공간에서 $\nabla$ 는 각 좌표 축에 대한 편미분 벡터 $(\partial U / \partial x, \partial U / \partial y)$ 를 의미하며, 이는 $U$ 가 가장 가파르게 증가하는 방향을 가리키는 벡터가 된다. 따라서 $-\nabla U_{att}$ 는 항상 목표 방향을 가리키며, 거리에 비례하는 크기를 갖는다.

### 2.2 Repulsive Potential

![Repulsive potential field](<APF/image (1).png>)

장애물에 대한 척력 potential은 영향 반경 $\rho_0$ 내에서만 작용하도록 정의된다:

$$
U_{rep}(x) = \begin{cases} \dfrac{1}{2} \eta \left( \dfrac{1}{\rho(x)} - \dfrac{1}{\rho_0} \right)^2 & \text{if } \rho(x) \leq \rho_0 \\[6pt] 0 & \text{if } \rho(x) > \rho_0 \end{cases}
$$

- $\rho(x) = \| x - x_{obs} \|$: 위치 $x$ 에서 가장 가까운 장애물까지의 거리
- $\rho_0$: 영향 반경 (influence distance) — 이보다 먼 장애물은 무시
- $\eta$: repulsive gain

#### 핵심 성질

- $\rho(x) \to 0$ 일 때 $U_{rep} \to \infty$ (충돌 방지)
- $\rho(x) = \rho_0$ 일 때 $U_{rep} = 0$ (영향권 경계에서 연속성 보장)
- $\rho(x) > \rho_0$ 일 때 $U_{rep} = 0$ (멀리 있는 장애물은 자동 무시)

반비례 형태 $1/\rho - 1/\rho_0$ 의 설계 의도는 두 가지이다. 첫째, $\rho \to 0$ 에서 발산하여 충돌을 무한히 큰 cost로 penalize한다. 둘째, $\rho = \rho_0$ 에서 정확히 0이 되어 영향권 안팎이 부드럽게 연결된다 (불연속 회피).

대응되는 repulsive force는 potential의 negative gradient로 주어진다:

$$
F_{rep}(x) = \eta \left( \frac{1}{\rho} - \frac{1}{\rho_0} \right) \frac{1}{\rho^2} \nabla \rho(x), \quad \text{if } \rho \leq \rho_0
$$

방향: 장애물에서 멀어지는 방향 ($\nabla \rho$ 가 그 단위 벡터를 제공).

### 2.3 Total Force와 다중 장애물 처리

![Total force with multiple obstacles](<APF/image (2).png>)

합산된 potential의 gradient가 로봇에 작용하는 총 force가 된다:

$$
F_{total}(x) = F_{att}(x) + \sum_i F_{rep,i}(x)
$$

다수의 장애물이 존재할 경우 각 장애물 $i$ 에 대한 척력을 벡터 합산한다. $\rho_0$ 밖의 장애물은 자동으로 0이 되어 계산에서 제외되므로, 영향 반경 cutoff는 계산 효율 측면에서도 이점을 제공한다.

## 3. Path Generation Procedure (표준 사용법)

APF 기반 path generation은 다음과 같이 동작한다:

```text
while ||F_total|| > epsilon and not goal_reached:
    F = F_att(x) + sum_i F_rep_i(x)
    x <- x + dt * F
    record path point x
```

종료 조건은 $F_{total} \approx 0$ (force 평형, 이상적으로는 goal 도달).

## 4. 알려진 한계 (Path Generation 사용 시)

APF를 path generator로 사용할 경우 다음의 본질적 한계가 존재한다 [Koren & Borenstein, 1991]:

### 4.1 Local Minima

다수 장애물의 척력 합이 인력과 정확히 상쇄되는 지점에서 $F_{total} = 0$ 이 되어 로봇이 정지한다. 특히 reversed-C 형태의 장애물 배치나 대칭적 장애물 분포에서 빈번하게 발생하며, goal 도달을 보장하지 못하는 가장 큰 원인이다.

### 4.2 Goal Non-Reachable with Obstacles Nearby (GNRON)

목표 지점이 장애물에 매우 가까운 경우, 목표 근처에서도 척력이 강하게 작용하여 목표 도달이 불가능해진다.

### 4.3 Oscillation in Cluttered Environments

다수의 장애물이 존재하는 환경에서 각 장애물의 척력 합 벡터가 위치에 따라 급격히 변하여 진동이 발생하며, 이는 불안정한 경로를 생성한다.

### 4.4 Oscillation in Narrow Passages

좁은 통로에서 양쪽 벽의 척력이 명목상 균형을 이루나, 작은 disturbance에도 한쪽 벽으로 쏠리며 반대쪽 벽에서 다시 밀려나는 지속적 진동이 발생할 수 있다.

위 한계들은 모두 APF가 *gradient-based path generator*로 사용될 때 발생하는 문제이다.

## 5. 본 연구 적용: Trajectory Evaluation으로의 변환

### 5.1 Motivation

본 연구의 hybrid 자율주행 시스템에서 rule-based planner와 E2E planner는 이미 candidate trajectory를 생성한다. Selector의 역할은 *trajectory 생성*이 아닌 *trajectory 평가*이다. 따라서 APF를 다음과 같이 재해석한다:

- **표준 사용**: $U(x)$ 의 gradient를 force로 변환하여 path를 생성
- **본 연구 사용**: $U_{rep}(x)$ 의 값 자체를 trajectory 점들에 누적하여 안전도 cost로 활용

### 5.2 Cost Formulation

Candidate trajectory $\tau = \{ p_1, p_2, \ldots, p_N \}$ 에 대한 안전도 cost를 다음과 같이 정의한다:

$$
J(\tau) = \sum_{p \in \tau} U_{rep}(p) = \sum_{p \in \tau} \frac{1}{2} \eta \left( \frac{1}{\rho(p)} - \frac{1}{\rho_0} \right)^2 \cdot \mathbb{1}[\rho(p) \leq \rho_0]
$$

여기서 $\rho(p)$ 는 trajectory 점 $p$ 에서 가장 가까운 장애물까지의 거리이다.

### 5.3 Selection Rule

두 candidate $\tau_{RB}$ (rule-based), $\tau_{E2E}$ (E2E) 중 다음 규칙으로 선택한다:

$$
\tau^* = \arg\min_{\tau \in \{\tau_{RB}, \tau_{E2E}\}} J(\tau), \quad \text{s.t. } \tau \text{ is collision-free}
$$

Collision-free 검증은 별도의 hard filter로 처리하며, $J(\tau)$ 는 collision-free 후보들 사이의 *안전 마진* 비교에 사용된다. 이는 hard safety constraint와 soft proximity scoring을 결합한 2-stage decision 구조이다.

### 5.4 Section 4 한계의 미적용

본 적용에서는 §4의 한계들이 모두 회피된다:

| 한계 | 본 연구 적용 시 | 사유 |
| :-- | :-- | :-- |
| Local minima | 발생 안 함 | Gradient descent 미사용 |
| GNRON | 발생 안 함 | Goal 도달은 selector 책임 아님 |
| Cluttered oscillation | 발생 안 함 | Force 합산 대신 cost 적분 |
| Narrow passage oscillation | 발생 안 함 | 동일 사유 |

이는 APF를 *경로 생성기*가 아닌 *경로 평가 함수*로 사용함으로써 얻는 구조적 이점이다.

### 5.5 인력 Potential의 미적용

본 selector는 $U_{att}$ 를 사용하지 않는다. 이유:

- Goal-directed motion은 candidate planner (rule-based, E2E)가 자체적으로 처리한다.
- Selector의 책임 영역은 안전성 평가에 한정되며, 이는 책임 분리 (separation of concerns) 원칙에 부합한다.
- 결과적으로 $K_{att}$ 파라미터가 제거된다. 또한 $J$ 가 두 candidate 간 *상대 비교*에만 사용되므로 $\eta$ 는 비교 결과에 영향을 주지 않아, 실질적으로 $\rho_0$ 단일 파라미터로 환원된다.

## 6. 파라미터 정리

| 기호 | 의미 | 본 연구 사용 | 시작값 |
| :-- | :-- | :-- | :-- |
| $K_{att}$ | 인력 gain | 미사용 | — |
| $\eta$ | 척력 gain | 명목상 사용 (비교 결과 무관) | 1.0 |
| $\rho_0$ | 영향 반경 | 핵심 파라미터 | 5.0 m |

## References

- Khatib, O. (1986). *Real-time obstacle avoidance for manipulators and mobile robots*. The International Journal of Robotics Research.
- Koren, Y., & Borenstein, J. (1991). *Potential field methods and their inherent limitations for mobile robot navigation*. ICRA.
