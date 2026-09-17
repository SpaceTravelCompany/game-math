---
title: 벡터 보간 (Vector Interpolation)
slug: vector-interpolation
---

## 소개

보간(Interpolation)은 두 값 사이를 부드럽게 채우는 기법이다. 게임에서 캐릭터의 이동, 카메라 추적, 애니메이션 블렌딩, UI 트랜지션 등 거의 모든 곳에서 사용된다.

> **핵심**
> Lerp는 직선 보간, Slerp는 구면 보간, Nlerp는 정규화된 직선 보간이다.

> **참고**: 스칼라(숫자) 값의 부드러운 전환에는 **Smoothstep(《삼각함수 활용》 참고)** 도 자주 쓰인다.

---

## 1. Lerp (Linear Interpolation)

두 벡터 사이를 직선으로 보간한다.

### 공식

$$
\text{lerp}(\mathbf{a}, \mathbf{b}, t) = \mathbf{a} + t \times (\mathbf{b} - \mathbf{a}) = (1 - t) \times \mathbf{a} + t \times \mathbf{b}
$$

여기서 $t$는 0~1 사이의 보간 인자다.

### 특징
- $t=0 \Rightarrow \mathbf{a}$, $t=1 \Rightarrow \mathbf{b}$
- Lerp 자체는 $t$에 대해 등속(직선 보간)이나, 방향 벡터에 쓰면 각속도가 균일하지 않다
- 계산이 가장 빠름

**게임에서의 활용:**
- **위치 이동**: 캐릭터를 A에서 B로 부드럽게 이동
- **색상 보간**: 두 색 사이의 그라데이션
- **카메라 추적**: 카메라가 타겟을 부드럽게 따라감

```text
// 카메라가 플레이어를 부드럽게 따라가기
t = 0.1  // 따라가는 속도 (0~1, 클수록 빠름)
camera.position = lerp(camera.position, targetPosition, t)
```

### 프레임 독립 Lerp

프레임 속도가 다른 환경에서 일관된 결과를 위해 지수 감쇠를 사용한다:

```text
// 프레임 의존적 (불안정)
camera.position = lerp(camera.position, target, 0.1)

// 프레임 독립적 (안정)
rate = 1 - pow(1 - 0.1, deltaTime × 60)  // 60fps 기준
camera.position = lerp(camera.position, target, rate)
```

프레임 독립(지수 감쇠)의 유도와 다른 보간법과의 비교는 **《지수 감쇠》 §1·§5** 참고.

---

## 2. Slerp (Spherical Linear Interpolation)

두 단위 벡터 사이를 **구면** 위에서 보간한다. 즉, 각도가 균일하게 변한다.

### 공식

$$
\theta = \arccos(\text{clamp}(\mathbf{a} \cdot \mathbf{b},\ -1,\ 1))
$$

$$
\text{slerp}(\mathbf{a}, \mathbf{b}, t) = \frac{\sin((1 - t)\theta)}{\sin\theta} \mathbf{a} + \frac{\sin(t\theta)}{\sin\theta} \mathbf{b}
$$

> **수치 안정성**: $\theta < 0.001$($\sin\theta \approx 0$)일 때는 $0$으로 나누기를 방지하기 위해 $\text{lerp}(\mathbf{a}, \mathbf{b}, t)$ 후 정규화(Nlerp)로 대체한다.  
> **정반대 방향 퇴화 ($\theta \approx \pi$, $\mathbf{a} \cdot \mathbf{b} \approx -1$)**: 두 벡터가 정반대를 가리킬 때는 두 벡터를 잇는 최단 호(대원)의 회전축이 무수히 많아 유일하게 결정되지 않는다. 이때는 $\mathbf{a}$에 수직인 임의의 축을 하나 선택하여 180° 회전해야 한다.  
> *(참고: $\mathbf{a} \cdot \mathbf{b} < 0$일 때 부호를 반전시켜 최단 경로를 취하는 규칙은 $\mathbf{q} \sim -\mathbf{q}$가 같은 회전을 나타내는 **쿼터니언 Slerp**의 성질이며, 3D 방향 벡터 Slerp에서는 부호를 뒤집으면 반대 방향으로 향하게 되므로 적용하지 않는다.)*

### 특징
- **각속도 일정**: 회전 보간에 적합
- 단위 벡터에서만 의미 있음
- lerp보다 계산 비용이 높음 ($\sin$, $\arccos$ 사용)
- 방향 보간에 사용 (위치 보간에는 부적합)

**게임에서의 활용:**
- **카메라 회전**: 타겟을 향해 회전할 때 각도 균일하게
- **애니메이션**: 본(Bone) 회전의 부드러운 블렌딩
- **광선 추적**: 광선 방향의 부드러운 변화

```text
// 적이 플레이어를 향해 부드럽게 회전
currentDir = normalize(enemy.forward)
targetDir = normalize(player.position - enemy.position)
t = 0.05
enemy.forward = slerp(currentDir, targetDir, t)
```

---

## 3. Nlerp (Normalized Lerp)

Lerp 후 정규화하는 방식이다. Slerp의 근사치이면서 계산이 더 빠르다.

### 공식

$$
\text{nlerp}(\mathbf{a}, \mathbf{b}, t) = \text{normalize}(\text{lerp}(\mathbf{a}, \mathbf{b}, t)) = \text{normalize}\big((1 - t) \times \mathbf{a} + t \times \mathbf{b}\big)
$$

### 특징
- 방향은 정확하지만 각속도는 일정하지 않음 (Slerp와 달리)
- Slerp보다 훨씬 빠름 ($\sin$, $\arccos$ 없음)
- 각도 차이가 클 때 약간 비선형적

### Slerp vs Nlerp 비교

| 특징 | Slerp | Nlerp |
|------|-------|-------|
| 각속도 | 일정 | 비선형 |
| 속도 | 느림 ($\sin$, $\arccos$) | 빠름 |
| 정확도 | 높음 | 근사치 |
| 사용 추천 | 정밀 회전 | 일반적인 회전 |

> **실전 팁**: 대부분의 게임에서는 Nlerp로 충분하다. 정밀한 각도 보간이 필요한 경우(예: 컷신, 시네마틱)에만 Slerp를 사용하자.

---

## 4. 이중 선형 보간 (Bilinear)

2D 공간에서 네 개의 점 사이를 보간한다.

$$
\text{result} = \text{lerp}\big(\text{lerp}(\mathbf{P}_{00}, \mathbf{P}_{10}, u),\ \text{lerp}(\mathbf{P}_{01}, \mathbf{P}_{11}, u),\ v\big)
$$

**게임에서의 활용:**
- **타일맵**: 텍스처 필터링 (바이리니어 필터)
- **지형 높이**: 하이트맵에서 위치의 높이 계산
- **색상 그라데이션**: 2D 색상 공간 보간

```text
// 하이트맵에서 플레이어 위치의 높이
gridX = floor(player.x / cellSize)
gridZ = floor(player.z / cellSize)
u = (player.x / cellSize) - gridX
v = (player.z / cellSize) - gridZ

h00 = heightMap[gridX][gridZ]
h10 = heightMap[gridX+1][gridZ]
h01 = heightMap[gridX][gridZ+1]
h11 = heightMap[gridX+1][gridZ+1]

height = bilinear(h00, h10, h01, h11, u, v)
```

---

## 5. 스무스댐프 (SmoothDamp)

Unity의 SmoothDamp와 같은 방식으로, 목표값에 부드럽게 도달하면서 관성을 시뮬레이션한다. 스프링·Lerp와의 상세 비교는 **《스프링 & 댐핑》 §5** 참고.

### 공식 (크리티컬 댐핑)

$$
\omega = \frac{2}{\text{smoothTime}}, \quad x = \omega \cdot dt, \quad \text{exp} = \frac{1}{1 + x + 0.48x^2 + 0.235x^3}
$$

$$
\text{change} = \text{current} - \text{target}
$$

$$
\text{temp} = (\text{velocity} + \omega \cdot \text{change}) \cdot dt
$$

$$
\text{velocity} = (\text{velocity} - \omega \cdot \text{temp}) \cdot \text{exp}
$$

$$
\text{output} = \text{target} + (\text{change} + \text{temp}) \cdot \text{exp}
$$

**게임에서의 활용:**
- **카메라 추적**: 타겟을 부드럽게 따라가면서 튕기지 않음
- **UI 애니메이션**: 메뉴가 슥 나타나고 멈춤
- **물리 보간**: 네트워크 플레이어 위치 보정

> **SmoothDamp vs Lerp**: Lerp는 지수 감쇠(1차 동역학)라 관성 없이 목표에 다가가는 속도가 점점 느려진다. SmoothDamp는 속도(관성) 상태를 쓰지만 크리티컬 댐핑($\zeta = 1$, 절 제목 참고)으로 설계되어, 속도가 부드럽게 줄면서 오버슈트 **없이** 목표에 도달한다 — 크리티컬 댐핑은 정의상 오버슈트가 없다. 오버슈트는 저감쇠 스프링의 특성이다.

---

## 6. Catmull-Rom 보간

> **참고**: 여러 곡선을 스플라인으로 이어붙이는 방식은 **《베지어 곡선》 문서 §7** 참고.

여러 점을 지나는 부드러운 곡선을 만든다. 각 점을 통과하면서 연속적인 곡선을 형성한다.

각 호출에서 $\mathbf{p}_1 \to \mathbf{p}_2$가 실제 곡선 세그먼트이고, $\mathbf{p}_0$, $\mathbf{p}_3$는 접선(방향) 조절점으로 곡선이 지나지 않는다.

### 공식

$$
\text{catmullRom}(\mathbf{p}_0, \mathbf{p}_1, \mathbf{p}_2, \mathbf{p}_3, t) = \frac{1}{2} \Big( 2\mathbf{p}_1 + (-\mathbf{p}_0 + \mathbf{p}_2)t + (2\mathbf{p}_0 - 5\mathbf{p}_1 + 4\mathbf{p}_2 - \mathbf{p}_3)t^2 + (-\mathbf{p}_0 + 3\mathbf{p}_1 - 3\mathbf{p}_2 + \mathbf{p}_3)t^3 \Big)
$$

**게임에서의 활용:**
- **카메라 경로**: 미리 정의된 경로를 따라 카메라 이동
- **레일 슈터**: 레일 위에서 플레이어/객체 이동
- **보행 경로**: NPC가 자연스러운 경로를 따라 걷기

---

## 7. 보간 방법 선택 가이드

| 상황 | 추천 | 이유 |
|------|------|------|
| 위치 이동 | Lerp | 빠르고 직선 이동에 적합 |
| 방향 회전 | Slerp/Nlerp | Slerp: 각도 균일 / Nlerp: 빠른 근사(각속도 비균일) |
| 카메라 추적 | SmoothDamp | 자연스러운 관성 |
| 컷신 회전 | Slerp | 정밀한 각속도 |
| 경로 따라가기 | Catmull-Rom | 곡선 통과 |
| 텍스처 필터링 | Bilinear | 2D 공간 보간 |
| 색상 변화 | Lerp | 빠르고 충분 |

---

## 8. 빠른 참조

| 함수 | 공식 | 용도 |
|------|------|------|
| Lerp | $\mathbf{a} + t(\mathbf{b} - \mathbf{a})$ | 위치, 색상 |
| Slerp | $\frac{\sin((1-t)\theta)}{\sin\theta} \mathbf{a} + \frac{\sin(t\theta)}{\sin\theta} \mathbf{b}$ | 회전, 방향 |
| Nlerp | $\text{normalize}(\text{lerp}(\mathbf{a}, \mathbf{b}, t))$ | 빠른 회전 |
| SmoothDamp | 크리티컬 댐핑 | 카메라, UI |
| Catmull-Rom | 3차 다항식 | 경로, 곡선 |
| Bilinear | $\text{lerp}(\text{lerp}(\mathbf{P}_{00}, \mathbf{P}_{10}, u), \text{lerp}(\mathbf{P}_{01}, \mathbf{P}_{11}, u), v)$ | 2D 공간 |