---
title: 충돌 검출 (Collision Detection)
slug: collision-detection
---

## 소개

충돌 검출은 두 객체가 물리적으로 겹치는지 판단하는 기술이다. 물리 엔진의 핵심이며, 캐릭터 이동, 발사체 판정, 환경 상호작용 등 게임의 거의 모든 시스템에서 사용된다.

> **핵심**
> Broad Phase(빠른 거르기) → Narrow Phase(정밀 검사) → 충돌 응답(물리 반응).

> **법선 관례**
> 이 문서는 충돌 법선 $\mathbf n$을 **A에서 B 방향**으로 사용한다 (예: 구-구에서 $\mathbf n = (\mathbf C_b - \mathbf C_a)/\text{dist}$). 《충돌처리》는 B→A 관례이므로 두 문서를 함께 읽을 때 법선 부호 반전에 주의하라.

---

## 1. 충돌 검출 파이프라인

```text
1. Broad Phase (브로드 페이즈)
   → AABB/구로 빠르게 가능한 충돌 쌍 추리기
   → 대부분의 쌍을 여기서 제거

2. Narrow Phase (내로우 페이즈)
   → 정확한 형태로 충돌 확인
   → SAT, GJK, 삼각형 메시 검사

3. Collision Response (충돌 응답)
   → 충돌법선, 관통 깊이 계산
   → 위치 보정, 속도 반사, 힘 적용
```

---

## 2. AABB (Axis-Aligned Bounding Box)

축에 정렬된 박스. 가장 빠르고 단순한 충돌 형태다.

### AABB 정의

$$\text{AABB} = \{ \min: (\min_x, \min_y, \min_z),\; \max: (\max_x, \max_y, \max_z) \}$$

### AABB vs AABB 충돌

$$\min_x^a \le \max_x^b \;\land\; \max_x^a \ge \min_x^b \;\land\; \min_y^a \le \max_y^b \;\land\; \max_y^a \ge \min_y^b \;\land\; \min_z^a \le \max_z^b \;\land\; \max_z^a \ge \min_z^b$$

### 충돌법선과 관통 깊이

어느 축으로 가장 적게 겹쳤는지 확인:

$$\text{overlap}_x = \min(\max_x^a, \max_x^b) - \max(\min_x^a, \min_x^b)$$

$$\text{overlap}_y = \min(\max_y^a, \max_y^b) - \max(\min_y^a, \min_y^b)$$

$$\text{overlap}_z = \min(\max_z^a, \max_z^b) - \max(\min_z^a, \min_z^b)$$

가장 작은 오버랩이 충돌 방향:

$$\text{normal} = \begin{cases} (\pm 1, 0, 0) & \text{if } \text{overlap}_x \text{ is smallest} \\ (0, \pm 1, 0) & \text{if } \text{overlap}_y \text{ is smallest} \\ (0, 0, \pm 1) & \text{if } \text{overlap}_z \text{ is smallest} \end{cases}$$

### 장단점

| 장점 | 단점 |
|------|------|
| 매우 빠름 | 회전 불가 (축 정렬 필수) |
| 구현 간단 | 회전 객체에 부정확 |
| 메모리 적음 | 객체 회전 시 AABB 재계산 필요 |

---

## 3. Sphere 충돌

### Sphere vs Sphere

$$\mathbf{d} = \mathbf{C}_b - \mathbf{C}_a$$

$$\text{distSq} = \mathbf{d} \cdot \mathbf{d}$$

$$\text{radiusSum} = r_a + r_b$$

$$\text{distSq} < \text{radiusSum}^2 \quad \Rightarrow \quad \text{hit}$$

충돌 시:

$$\text{dist} = \sqrt{\text{distSq}}$$

$$\mathbf{n} = \frac{\mathbf{d}}{\text{dist}}$$

$$\text{depth} = \text{radiusSum} - \text{dist}$$

> **최적화**: sqrt를 피하기 위해 제곱 거리로 먼저 비교한다.

### Sphere vs AABB

AABB 내부의 구 중심에 가장 가까운 점:

```text
closestX = clamp(sphere.center.x, box.min.x, box.max.x)
closestY = clamp(sphere.center.y, box.min.y, box.max.y)
closestZ = clamp(sphere.center.z, box.min.z, box.max.z)

closest = (closestX, closestY, closestZ)
d = sphere.center - closest
distSq = dot(d, d)

if distSq < sphere.radius²:
    dist = sqrt(distSq)
    normal = d / dist
    depth = sphere.radius - dist
    return hit(normal, depth)
return no hit
```

> **주의 — 중심이 박스 내부인 퇴화 케이스**
> 구 중심이 AABB 안에 있으면 clamp 결과가 중심 자체라 `closest == center`, 즉 `d = 0`이 되어 `normal = d / dist`가 $0/0$(NaN)이 된다. 중심이 박스 내부일 때는 법선이 정의되지 않으므로, 세 축 중 가장 적게 관통한 축을 골라 가장 가까운 면 방향으로 밀어내는 **별도 분기**(최소 관통 축)가 필요하다 (Ericson, *Real-Time Collision Detection* §5.2.2).

**게임에서의 활용:**
- **캐릭터 충돌**: 인간형 캐릭터를 구 또는 캡슐로 표현
- **발사체**: 총알, 화살 등을 구로 처리
- **트리거**: 특정 영역 진입 감지

---

## 4. OBB (Oriented Bounding Box)

회전 가능한 박스다. AABB보다 정확하지만 계산이 복잡하다.

### OBB 정의

$$\text{OBB} = \{ \mathbf{c}: (c_x, c_y, c_z),\; \text{axes}: [\mathbf{r}, \mathbf{u}, \mathbf{f}],\; \mathbf{h}: (h_x, h_y, h_z) \}$$

$\mathbf{r}, \mathbf{u}, \mathbf{f}$는 3개의 직교 단위 벡터 (right, up, forward), $\mathbf{h}$는 halfSize.

### OBB vs OBB: SAT 사용

SAT(Separating Axis Theorem)를 사용한다. (다음 섹션 참조)

---

---

## 5. SAT (Separating Axis Theorem, 분리축 정리)

두 볼록(convex) 다포체 사이에 **"둘을 가르는 직선(또는 평면)"이 단 하나라도 존재하면 두 물체는 충돌하지 않는다**는 정리다.

### 손전등과 그림자 직관

두 물체 사이에 여러 각도로 손전등을 비춘다고 상상해보자:

```text
       물체 A         물체 B
       ┌───┐          ▲
       └───┘         / \
                    └───┘
  ─────────────────────────────────── (축에 투영)
       [---]          [---]   <-- 그림자가 서로 떨어져 있음! (분리축 발견 → 충돌 아님)
```

- 특정 각도로 비췄을 때 바닥에 맺힌 **두 그림자가 서로 떨어져 있다면(overlap = 0)**, 두 물체 사이에는 빈 틈이 있는 것이므로 **절대 충돌하지 않는다**.
- 반대로 **가능한 모든 각도에서 그림자가 전부 겹칠 때만** 비로소 두 물체가 진짜로 충돌한 것이다.

### 검사해야 할 후보 축

무한한 각도를 다 검사할 필요 없이, 기하학적으로 의미 있는 축만 검사하면 된다:

1. **AABB vs AABB**: 3개 축만 검사 (X, Y, Z축)
2. **OBB vs OBB**: 총 **15개 축**을 검사
   - 물체 A의 로컬 3축 (A의 면에 수직인 축)
   - 물체 B의 로컬 3축 (B의 면에 수직인 축)
   - A의 모서리와 B의 모서리의 외적 $3 \times 3 = 9$개 축 (모서리끼리 부딪히는 모서리-모서리 접촉 판정용)

```text
for axis in candidateAxes:
    projA = project(A, axis) // [minA, maxA]
    projB = project(B, axis) // [minB, maxB]

    // 하나라도 그림자가 떨어져 있으면 즉시 조기 종료!
    if not overlap(projA, projB):
        return no hit

// 15개 축 모두에서 그림자가 겹침 → 충돌!
return hit
```

> **크로스 축 평행 퇴화 주의**  
> A의 모서리와 B의 모서리가 거의 평행하면 $\mathbf{A}_i \times \mathbf{B}_j \approx \mathbf{0}$이 되어 축의 길이가 0이 된다. 이 축은 무의미하므로 외적 벡터 길이가 $\varepsilon$ 미만이면 검사에서 제외한다.

### 최소 관통 축 (MTV — Minimum Translation Vector)

모든 축 중에서 **그림자가 가장 적게 겹친 축**이 바로 두 물체를 떼어놓기 위해 가장 적은 힘으로 밀어낼 수 있는 최단 탈출 경로(MTV)다:

$$\text{MTV} = \mathbf{n}_{\min} \cdot \text{minOverlap}$$

---

## 6. GJK (Gilbert–Johnson–Keerthi) 알고리즘

SAT는 다면체의 면/모서리가 많아지면 검사해야 할 축의 수가 급증한다.  
**GJK**는 구, 캡슐, 원기둥, 임의의 볼록 다면체 등 **어떤 형태의 볼록체든 지원하는 현대 3D 물리 엔진(Bullet, PhysX)의 핵심 알고리즘**이다.

### 핵심 아이디어: 민코프스키 차이 (Minkowski Difference)

두 물체 $A$와 $B$가 겹친다는 것은, $A$ 안의 점 $\mathbf{a}$와 $B$ 안의 점 $\mathbf{b}$가 같아지는 점($\mathbf{a} = \mathbf{b}$)이 존재한다는 뜻이다.  
즉, $\mathbf{a} - \mathbf{b} = \mathbf{0}$ 이다!

$$A \ominus B = \{ \mathbf{a} - \mathbf{b} \mid \mathbf{a} \in A,\; \mathbf{b} \in B \}$$

> **"두 물체 $A, B$가 충돌하는가?"**  
> $\iff$ **"민코프스키 차이 도형 $A \ominus B$가 원점 $(0,0,0)$을 포함하는가?"**

```text
    물체 A 와 B의 충돌 판정                민코프스키 차이 공간 (A - B)
       ┌───┐                                      ┌──────┐
       │ A ├───┐                                  │      │
       └───┤ B │   ======(변환)=======>           │  (0,0)  <-- 원점이 도형 안!
           └───┘                                  │      │    (즉, 충돌!)
                                                  └──────┘
```

복잡한 두 물체의 상호작용이 **"도형 하나가 원점을 품고 있는가?"**라는 초간단 포함 문제로 바뀐다!

### Support 함수: 외곽 끝점만 빠르게 구하기

민코프스키 차이 도형 전체를 컴퓨터 메모리에 다 만들 필요가 없다.  
주어진 방향 $\mathbf{d}$로 가장 멀리 뻗은 끝점 하나만 구하면 된다:

$$\text{support}(A \ominus B, \mathbf{d}) = \text{furthestPoint}(A, \mathbf{d}) - \text{furthestPoint}(B, -\mathbf{d})$$

$A$에서 $\mathbf{d}$ 방향 맨 끝 점을 찾고, $B$에서 반대 방향 $-\mathbf{d}$ 맨 끝 점을 찾아 빼면 끝이다.

### 심플렉스(Simplex)로 원점 포위하기

GJK는 민코프스키 차이 도형 안에서 **가장 단순한 기하 단위(심플렉스)**를 점진적으로 만들어 원점을 포위한다:
- **1개 점** (점) $\to$ **2개 점** (선분) $\to$ **3개 점** (삼각형) $\to$ **4개 점** (사면체)

```text
1. 임의의 방향 d로 첫 번째 점을 찾는다.
2. 원점 방향 (-점)으로 두 번째 점을 찾아 선분을 만든다.
3. 선분에서 원점을 향하는 법선 방향으로 세 번째 점을 찾아 삼각형을 만든다.
4. 삼각형에서 원점을 향해 네 번째 점을 찾아 사면체를 만든다.
5. 사면체가 원점 (0,0,0)을 완전히 둘러싸면 → "충돌(Hit)!"
6. 둘러싸지 못하고 더 이상 원점 쪽으로 나아갈 점이 없으면 → "충돌 아님(No Hit)!"
```

```text
function GJK(A, B):
    direction = (1, 0, 0)    // 임의 초기 방향
    simplex = [support(A, B, direction)]

    direction = -simplex[0]

    while true:
        point = support(A, B, direction)
        if dot(point, direction) < 0:
            return no hit    // 원점을 지나지 않음 → 충돌 없음

        simplex.add(point)
        if containsOrigin(simplex, direction):
            return hit        // 원점 포함 → 충돌
```

`containsOrigin(simplex, direction)`은 현재 심플렉스의 형태(선분/삼각형/사면체)에 따라 원점을 포함할 수 없는 영역을 버리고, **원점을 향하는 새 탐색 방향 `direction`을 갱신**한다. 사면체가 원점을 완전히 감싸면 충돌을 보고한다. 이 방향 갱신이 GJK 반복을 수렴시키는 핵심 단계다.

### Support 함수

$A \ominus B$에서 direction 방향으로 가장 먼 점:

```text
function support(A, B, direction):
    pA = furthestPoint(A, direction)
    pB = furthestPoint(B, -direction)
    return pA - pB
```

> **직관**: Support($A, B, d$)는 $M$에서 방향 $d$로 가장 먼 점이다.
> $A$의 $d$ 방향 끝점과 $B$의 $-d$ 방향 끝점의 차이
> ($\text{support}(A, d) - \text{support}(B, -d)$)로 계산하므로,
> **각 물체의 '외곽 끝점'을 빼면 Minkowski 차이의 외곽 끝점**이 나온다.

### 장점

- 컨벡스 형태라면 무엇이든 처리 가능 (구, 박스, 컨벡스 헐)
- 빠른 조기 종료 (보통 수 회 반복으로 결정)
- 충돌 여부만 필요할 때 매우 효율적

### 제한

- 컨벡스 형태만 지원 (컨케이브는 분해 필요)
- 충돌 깊이/법선은 EPA로 추가 계산

---

## 7. EPA (Expanding Polytope Algorithm)

GJK는 "충돌했는가?"(참/거짓)만 판정하고 끝난다.  
충돌이 확인되었을 때, **두 물체가 얼마나 파고들었는지(관통 깊이)와 어느 방향으로 밀어내야 하는지(충돌 법선)**를 구하는 알고리즘이 **EPA**다.

### 작동 원리 (심플렉스 부풀리기)

```text
       민코프스키 차이 도형
          ┌─────────────┐
          │  . (0,0)    │   <-- 원점을 품은 GJK 사면체에서 시작
          │   /\        │
          │  /__\       │   <-- 가장 가까운 면 쪽으로 새 점을 찾아 다면체를 확장
          └─────────────┘
```

1. GJK가 종료할 때 만든 사면체(심플렉스)를 초기 다면체로 삼는다.
2. 다면체의 모든 면 중에서 **원점 $(0,0,0)$과 가장 가까운 면**을 찾는다.
3. 그 면의 바깥 법선 방향으로 새 Support 점을 구한다.
4. 새로 구한 점이 기존 면보다 더 멀리 나아가지 않는다면($\approx$ 도형의 진짜 외곽 표면에 도달함), **그 면의 법선이 충돌 법선 $\mathbf{n}$, 원점까지의 거리가 관통 깊이 $d$** 가 된다!

---

## 8. 캡슐 충돌 (Capsule)

인간형 캐릭터에 가장 널리 쓰이는 충돌체다. 양 끝에 반구가 달린 원기둥 모양이다.

### 장점
- 세그먼트(선분) 하나와 반지름 $r$로 정의된다.
- **Capsule vs Sphere**: 구 중심에서 선분까지의 최단거리를 구한 뒤, $(r_{\text{sphere}} + r_{\text{capsule}})$과 비교하면 끝!
- **Capsule vs Capsule**: 두 선분 사이의 최단거리를 구한 뒤, 두 반지름의 합과 비교하면 끝!

---

## 9. 충돌 응답 (Collision Response 기초)

충돌을 검출한 뒤, 두 물체를 분리하고 물리적으로 반응시키는 단계다 (상세 내용은 《충돌처리》 문서 참고).

### 위치 보정 (Positional Correction)

겹쳐진 깊이($d$)만큼 두 물체를 질량의 역수에 비례하여 밀어낸다:

$$\text{moveRatio}_A = \frac{1/m_A}{1/m_A + 1/m_B}, \qquad \text{moveRatio}_B = \frac{1/m_B}{1/m_A + 1/m_B}$$

$$\Delta \mathbf{P}_A = -\mathbf{n} \cdot (d \times 0.8) \cdot \text{moveRatio}_A$$

$$\Delta \mathbf{P}_B = +\mathbf{n} \cdot (d \times 0.8) \cdot \text{moveRatio}_B$$

- 무거운 물체($m$이 큼 $\to 1/m$이 작음)는 조금만 밀리고, 가벼운 물체는 많이 밀린다.
- 정적 벽/바닥($m = \infty \to 1/m = 0$)은 전혀 밀리지 않는다.

### 속도 반사 (Velocity Reflection)

표면 바깥쪽 법선 $\hat{\mathbf{n}}$에 대해 입사 속도 $\mathbf{v}$를 반사한다 (반발 계수 $e$):

$$\mathbf{v}_{\text{new}} = \mathbf{v} - (1 + e)(\mathbf{v} \cdot \hat{\mathbf{n}})\hat{\mathbf{n}}$$

- $e = 0$: 완전 비탄성 (벽에 찰떡처럼 달라붙음)
- $e = 1$: 완전 탄성 (에너지 손실 없이 그대로 튕겨나감)

### 접촉점 (Contact Point)

충돌점을 기준으로 토크 계산:

$$\mathbf{r}_a = \text{contactPoint} - \mathbf{A}_{\text{COM}}$$

$$\mathbf{r}_b = \text{contactPoint} - \mathbf{B}_{\text{COM}}$$

$$\boldsymbol{\tau}_A = \mathbf{r}_a \times \mathbf{J}$$

$$\boldsymbol{\tau}_B = \mathbf{r}_b \times (-\mathbf{J})$$

---

## 10. 빠른 참조

| 형태 | 복잡도 | 용도 | 비고 |
|------|--------|------|------|
| AABB vs AABB | O(1) | 브로드페이즈 | 축 정렬만 |
| Sphere vs Sphere | O(1) | 발사체 | 가장 빠름 |
| Sphere vs AABB | O(1) | 캐릭터-벽 | clamp 활용 |
| OBB vs OBB (SAT) | O(15) | 회전 객체 | 15축 검사 |
| GJK | O(n) | 컨벡스 일반 | 조기 종료 |
| EPA | O(n) | GJK 후 깊이 | 수렴 |
| Capsule | O(1) | 캐릭터 | 세그먼트 거리 |
| Triangle Mesh | O(triangles) | 정밀 | 공간 분할 필요 |

| 단계 | 목적 | 방법 |
|------|------|------|
| Broad | 빠른 거르기 | AABB, Sweep & Prune |
| Narrow | 정밀 검사 | SAT, GJK |
| Response | 물리 반응 | MTV, Impulse |
| Continuous | 터널링 방지 | Ray/Sweep cast |