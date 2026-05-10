# Artificial Potential Field (APF)

목표점은 인력(attractive force), 장애물은 척력(repulsive force)을 발생시켜
로봇/차량을 합력의 방향으로 움직이게 하는 반응형 로컬 경로계획 기법.

## 개요

- **Attractive potential**: 목표점이 가까워질수록 작아지는 포텐셜
- **Repulsive potential**: 장애물에 가까워질수록 커지는 포텐셜
- **합성 포텐셜**의 음의 그래디언트 방향으로 이동

## 장단점

- 장점: 계산이 가볍고 실시간 동작 적합
- 단점: local minima에 빠질 수 있음 → global planner와의 결합 필요

## 참고

- O. Khatib, "Real-time obstacle avoidance for manipulators and mobile robots," 1986
