---
title: 베지어 곡선 (Bézier Curves)
slug: bezier-curves
---

## 소개

베지어 곡선은 제어점(control points)으로 정의되는 부드러운 곡선이다. 게임에서 카메라 경로, 애니메이션 보간, UI 애니메이션, 드로잉 툴, 그리고 글꼴 렌더링 등에 광범위하게 사용된다.

> **핵심**
> 제어점을 당기면 곡선이 따라간다. De Casteljau 알고리즘으로 재귀적으로 계산한다.

---

## 1. 선형 베지어 곡선 (1차)

두 점 사이의 직선이다.

$$B(t) = (1 - t) \cdot \mathbf{P_0} + t \cdot \mathbf{P_1} = \text{lerp}(\mathbf{P_0}, \mathbf{P_1}, t)$$

$$t \in [0, 1]$$

가장 기본적인 형태이며, Lerp와 동일하다.

---

## 2. 2차 베지어 곡선 (Quadratic)

세 개의 제어점으로 정의된다.

$$B(t) = (1 - t)^2 \cdot \mathbf{P_0} + 2(1 - t)t \cdot \mathbf{P_1} + t^2 \cdot \mathbf{P_2}$$

### 전개

$$B(t) = (1 - t) \cdot \text{lerp}(\mathbf{P_0}, \mathbf{P_1}, t) + t \cdot \text{lerp}(\mathbf{P_1}, \mathbf{P_2}, t)$$

### 특징

- P0에서 시작, P2에서 끝
- P1이 곡선을 "끌어당기는" 방향 역할
- P0와 P2에서의 접선은 P1을 향함

**게임에서의 활용:**
- **점수 팝업 곡선**: 직선 대신 곡선으로 자연스럽게
- **투사체 경로**: 화살이 살짝 휘어지는 경로
- **UI 애니메이션**: 메뉴가 곡선을 따라 이동

---

## 3. 3차 베지어 곡선 (Cubic)

네 개의 제어점으로 정의된다. 가장 많이 사용되는 베지어 곡선이다.

$$B(t) = (1 - t)^3 \cdot \mathbf{P_0} + 3(1 - t)^2 t \cdot \mathbf{P_1} + 3(1 - t) t^2 \cdot \mathbf{P_2} + t^3 \cdot \mathbf{P_3}$$

### 특징

- P0에서 시작, P3에서 끝
- P1, P2가 곡선 형태를 제어
- P0에서의 접선은 P1 방향, P3에서의 접선은 P2 방향

$$B'(0) = 3(\mathbf{P_1} - \mathbf{P_0}) \quad \text{시작점 방향}$$

$$B'(1) = 3(\mathbf{P_3} - \mathbf{P_2}) \quad \text{끝점 방향}$$

**게임에서의 활용:**
- **카메라 경로**: 시네마틱 카메라 이동
- **캐릭터 점프**: 포물선보다 더 자연스러운 곡선
- **드로잉**: 펜 툴에서 곡선 그리기
- **CSS 애니메이션**: cubic-bezier(p1x, p1y, p2x, p2y)

---

## 4. De Casteljau 알고리즘

베지어 곡선을 재귀적으로 계산하는 안정적인 방법이다.

### 원리

```text
function deCasteljau(points, t):
    if len(points) == 1:
        return points[0]

    newPoints = []
    for i in range(len(points) - 1):
        newPoints.append(lerp(points[i], points[i+1], t))

    return deCasteljau(newPoints, t)
```

### 3차 예시

> 4개 제어점 → 3개 중간점 → 2개 → 1개 (곡선 위의 점)

$$\mathbf{A} = \text{lerp}(\mathbf{P_0}, \mathbf{P_1}, t)$$

$$\mathbf{B} = \text{lerp}(\mathbf{P_1}, \mathbf{P_2}, t)$$

$$\mathbf{C} = \text{lerp}(\mathbf{P_2}, \mathbf{P_3}, t)$$

$$\mathbf{D} = \text{lerp}(\mathbf{A}, \mathbf{B}, t)$$

$$\mathbf{E} = \text{lerp}(\mathbf{B}, \mathbf{C}, t)$$

$$\mathbf{F} = \text{lerp}(\mathbf{D}, \mathbf{E}, t) \quad \text{최종 곡선 위의 점 } B(t)$$

### 장점

- 수치적으로 안정적 (Bernstein 다항식보다)
- 기하학적으로 직관적
- 곡선을 시각적으로 구성하는 과정이 이해하기 쉬움

> **실전 팁**: Bernstein 다항식 형태보다 De Casteljau가 부동소수점 오차가 적어 게임에서 더 추천된다.

---

## 5. 베지어 곡선의 속도

### 비균일 속도

베지어 곡선은 t가 일정하게 변해도 곡선 위의 이동 속도가 일정하지 않다:

- $t = 0.5$일 때 곡선의 중간점이 아님
- 제어점에 가까울수록 속도가 느려짐

### 균일 속도를 위한 호 길이 매개변수화 (Arc-Length Parameterization)

```text
// 1. 샘플링으로 누적 호 길이 테이블 생성
lengths = [0]
for i in 1..N:
    p1 = bezier(i / N)
    p2 = bezier((i-1) / N)
    lengths.append(lengths[-1] + distance(p1, p2))

// 2. 원하는 거리에 해당하는 t 찾기 (이진 탐색)
function tForDistance(targetDist):
    // lengths에서 targetDist 위치를 이진 탐색
    // 선형 보간으로 정확한 t 추정
```

**게임에서의 활용:**
- **레일 슈터**: 일정한 속도로 레일 따라 이동
- **경로 애니메이션**: 균일한 속도로 캐릭터 이동
- **진행 바**: 시각적 진행과 실제 진행 일치

---

## 6. 스플라인 (Spline)

여러 베지어 곡선을 이어서 부드러운 긴 곡선을 만든다.

### C0 연속성 (위치만 연속)

```text
// 곡선 A의 끝점 = 곡선 B의 시작점
A.P3 = B.P0

// C0는 위치 연속만 보장 — 접선 방향까지 이어지려면 G1(또는 C1) 이상 필요
```

### C1 연속성 (위치 + 1차 미분)

```text
// 끝점 일치 + 접선 방향/크기 일치
A.P3 = B.P0
B.P1 = A.P3 + (A.P3 - A.P2)    // 반사

// 부드럽게 이어짐
```

### C2 연속성 (위치 + 1차 + 2차 미분)

> 곡률까지 일치 → 매우 부드러움. 3차 베지어에서 C2는 추가 제약 필요

---

## 7. Catmull-Rom 스플라인

> **참고**: Catmull-Rom 보간 공식과 게임 활용은 **《벡터 보간》 문서 §6**에서도 다루므로 참고.

모든 제어점을 지나는 스플라인이다. 게임에서 베지어보다 더 자주 쓰인다.

### 변환: Catmull-Rom → 3차 베지어

> Catmull-Rom 점 $\mathbf{P_0}, \mathbf{P_1}, \mathbf{P_2}, \mathbf{P_3}$ (구간 $\mathbf{P_1} \sim \mathbf{P_2}$), 3차 베지어 제어점:

$$\mathbf{B_0} = \mathbf{P_1}$$

$$\mathbf{B_1} = \mathbf{P_1} + \frac{\mathbf{P_2} - \mathbf{P_0}}{6}$$

$$\mathbf{B_2} = \mathbf{P_2} - \frac{\mathbf{P_3} - \mathbf{P_1}}{6}$$

$$\mathbf{B_3} = \mathbf{P_2}$$

### 장점

- 모든 점을 통과 (베지어는 제어점을 안 지남)
- 제어점이 곡선 형태를 직접적으로 결정
- C1 연속성 자동 보장

**게임에서의 활용:**
- **카메라 경로**: 미리 찍은 점들을 부드럽게 연결
- **AI 경로**: 웨이포인트 사이 부드러운 이동
- **트랙**: 레이싱 게임 트랙 곡선
- **로프/밧줄**: 자연스럽게 늘어진 형태

---

## 8. 베지어 곡선의 미분 (속도/접선)

$$B'(t) = 3(1 - t)^2 (\mathbf{P_1} - \mathbf{P_0}) + 6(1 - t)t (\mathbf{P_2} - \mathbf{P_1}) + 3t^2 (\mathbf{P_3} - \mathbf{P_2})$$

$$B'(0) = 3(\mathbf{P_1} - \mathbf{P_0})$$

$$B'(1) = 3(\mathbf{P_3} - \mathbf{P_2})$$

$$\text{tangent} = \text{normalize}(B'(t))$$

**게임에서의 활용:**
- **카메라 방향**: 경로를 따라가며 바라보는 방향
- **오브젝트 회전**: 곡선 방향에 맞춰 객체 회전
- **궤적 표시**: 화살이 날아가는 경로 표시

---

## 9. 실전 예시: 투사체 경로

```text
// 3차 베지어로 화살 경로 계산
P0 = archer.position
P3 = target.position
P1 = P0 + (0, 5, 0)         // 시작에서 위로
P2 = P3 + (0, 5, 0)         // 끝에서 위로

// 경로 따라 이동
t += dt / flightTime
position = cubicBezier(P0, P1, P2, P3, t)

// 방향 (접선)
direction = normalize(bezierDerivative(P0, P1, P2, P3, t))
arrow.rotation = lookAt(direction)
```

---

## 10. 실전 예시: 카메라 시네마틱 경로

```text
// 제어점들로 경로 정의
controlPoints = [
    (0, 10, -20),   // 시작
    (0, 15, -10),   // 상승
    (5, 12, 0),     // 중간
    (0, 8, 20)      // 종료
]

// Catmull-Rom 스플라인으로 부드러운 경로
function getCameraPosition(t):
    // t를 세그먼트 인덱스와 로컬 t로 분할
    // N = len(controlPoints) (제어점 개수; 세그먼트 수 = N - 3)
    segIndex = floor(t × (N-1))
    localT = t × (N-1) - segIndex

    P0 = controlPoints[segIndex - 1]
    P1 = controlPoints[segIndex]
    P2 = controlPoints[segIndex + 1]
    P3 = controlPoints[segIndex + 2]

    return catmullRom(P0, P1, P2, P3, localT)
```

---

## 11. 빠른 참조

| 곡선 | 제어점 | 공식 | 활용 |
|------|--------|------|------|
| 1차 (선형) | 2 | $(1-t)\mathbf{P_0} + t\mathbf{P_1}$ | Lerp |
| 2차 | 3 | $(1-t)^2\mathbf{P_0} + 2(1-t)t\mathbf{P_1} + t^2\mathbf{P_2}$ | 간단한 곡선 |
| 3차 | 4 | $(1-t)^3\mathbf{P_0} + 3(1-t)^2t\mathbf{P_1} + 3(1-t)t^2\mathbf{P_2} + t^3\mathbf{P_3}$ | 범용 |
| N차 | N+1 | Bernstein 다항식 | 거의 안 씀 |

| 연속성 | 조건 | 효과 |
|--------|------|------|
| C0 | 끝점=시작점 | 위치만 연결 |
| C1 | C0 + 접선 일치 | 부드러운 연결 |
| C2 | C1 + 곡률 일치 | 매우 부드러움 |

| 알고리즘 | 장점 | 용도 |
|----------|------|------|
| Bernstein | 직접 공식 | 단순 계산 |
| De Casteljau | 수치 안정 | 게임 추천 |
| 호 길이 매개변수화 | 균일 속도 | 레일, 경로 |