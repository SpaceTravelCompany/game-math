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

## 5. SAT (Separating Axis Theorem)

두 컨벡스(convex) 형태가 분리될 수 있는 축이 하나라도 있으면 충돌하지 않는다는 정리다.

### 원리

모든 후보 축에 대해 두 객체의 투영이 겹치는지 검사. 하나라도 분리되면 충돌 없음.

```text
for axis in candidateAxes:
    projA = project(A, axis)
    projB = project(B, axis)
    if not overlap(projA, projB):
        return no hit        // 분리 축 발견!

return hit        // 모든 축에서 겹침 → 충돌
```

### AABB vs AABB 후보 축

$$\text{axes} = \{(1,0,0),\; (0,1,0),\; (0,0,1)\} \quad \text{(3개 축만 검사)}$$

### OBB vs OBB 후보 축

각 OBB의 3개 축 × 2 + 크로스 프로덕트 9개 = 15개 축:

$$\text{axes} = \{ \mathbf{A}_r, \mathbf{A}_u, \mathbf{A}_f, \mathbf{B}_r, \mathbf{B}_u, \mathbf{B}_f \}$$

$$\cup \; \{ \mathbf{A}_i \times \mathbf{B}_j \mid i \in \{r, u, f\},\; j \in \{r, u, f\} \}$$

> **주의 — 크로스 축 평행 퇴화**
> 두 축쌍이 (거의) 평행하면 $\mathbf{A}_i \times \mathbf{B}_j \approx \mathbf{0}$이 되어 투영 길이가 0인 무의미한 축이 된다. 길이가 $\varepsilon$ 미만인 크로스 축은 평행 퇴화로 간주해 스킵한다 (Ericson, *Real-Time Collision Detection* §4.4).

### 최소 관통 축 (MTV — Minimum Translation Vector)

$$\text{overlap} = \min(\text{projA}_{\max}, \text{projB}_{\max}) - \max(\text{projA}_{\min}, \text{projB}_{\min})$$

- $\text{overlap} \le 0$: 분리 축 → no hit
- 최소 overlap인 축이 MTV:

$$\text{MTV} = \mathbf{n}_{\min} \cdot \text{minOverlap}$$

이만큼 밀어내면 충돌 해결.

---

## 6. GJK (Gilbert–Johnson–Keerthi)

임의의 컨벡스 형태 사이의 충돌을 반복적으로(점진적으로) 검사하는 알고리즘이다. 충돌 확인 또는 탐색 방향으로 더 이상 원점에 접근할 수 없음을 판정하면 유한 번의 반복 후 종료한다.

> **Minkowski 차이의 직관**: 두 물체 $A$, $B$가 겹치는지 직접 따지려면 모든 점 쌍을
> 검사해야 하므로 어렵다. 대신 $B$를 원점을 중심으로 뒤집어($-B$) $A$에 더한 집합
> $M = A - B$를 만든다. 그러면 **"$A$와 $B$가 겹치는가?"**라는 문제가
> **"원점 $(0,0,0)$이 $M$ 안에 있는가?"**로 단순해진다 — 점 집합 간 연산이
> 점-집합 포함 관계 하나로 축소된 것이다.
>
> 1차원 예: $A = [2, 5]$, $B = [3, 6]$일 때
> $M = A - B = [2-6,\; 5-3] = [-4, 2]$, 원점 $0 \in M$이므로 충돌.
> 반면 $A = [2, 4]$, $B = [5, 7]$이면 $M = [-5, -1]$, 원점 밖 → 충돌 아님.

### 핵심 아이디어: 민코프스키 차이 (Minkowski Difference)

$$\mathbf{M} = A \ominus B = \{ \mathbf{a} - \mathbf{b} \mid \mathbf{a} \in A,\; \mathbf{b} \in B \}$$

$M$이 원점을 포함하면 충돌! → 원점이 $M$ 안에 있는지 검사.

### 심플렉스(Simplex) 구축

GJK는 원점이 $M$ 안에 있는지 확인하기 위해
점 → 선분 → 삼각형 → 사면체 순으로 점점 더 정밀한 심플렉스를 만들어가며
원점을 감싸는지 검사한다. 한 번에 사면체를 만들 수 없으니
**가장 유망한 점부터 하나씩 추가하며 점진적으로 좁혀가는** 방식이다.

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

GJK로 충돌을 확인한 후, 관통 깊이와 법선을 구하는 알고리즘이다.

> **EPA의 직관**: GJK가 충돌만 알려줬다면, EPA는 그 심플렉스에서 시작해
> 다면체를 바깥(면 법선 방향)으로 확장하며 원점에 가장 가까운 면을 갱신한다.
> 이 면의 법선이 충돌 법선, 원점까지의 거리가 관통 깊이(penetration depth)가 된다.
> 관통 깊이는 두 물체를 분리하기 위해 밀어내야 할 최소 거리다.

### 원리

GJK의 마지막 심플렉스(사면체)를 초기 다면체로 시작. 다면체를 바깥으로 확장하면서 원점에 가장 가까운 면을 갱신해 나감.

```text
function EPA(simplex, A, B):
    while true:
        // 가장 가까운 면 찾기
        face = closestFaceToOrigin(simplex)
        normal = face.normal
        distance = face.distance

        // 새 점 추가
        point = support(A, B, normal)
        d = dot(point, normal)

        if d - distance < ε:
            // 수렴 → 충돌 법선과 깊이
            return hit(normal, distance)

        // 면에 새 점 추가하여 심플렉스 확장
        simplex.expand(point)
```

---

## 8. 캡슐 충돌 (Capsule)

캐릭터 충돌에 가장 많이 쓰이는 형태다.

### 정의

$$\text{Capsule} = \{ \text{segment}: (\mathbf{A}, \mathbf{B}),\; \text{radius}: r \}$$

양끝이 구로 캡핑된 원기둥.

### Capsule vs Sphere

구의 중심에서 세그먼트까지의 최단 거리:

$$\mathbf{d} = \mathbf{C}_{\text{sphere}} - \text{closestPoint}(\mathbf{C}_{\text{sphere}}, \mathbf{A}, \mathbf{B})$$

$$\text{distSq} = \mathbf{d} \cdot \mathbf{d}$$

$$\text{distSq} < (r_{\text{sphere}} + r_{\text{capsule}})^2 \quad \Rightarrow \quad \text{hit}$$

충돌 시:

$$\text{dist} = \sqrt{\text{distSq}}, \quad \mathbf{n} = \frac{\mathbf{d}}{\text{dist}}, \quad \text{depth} = (r_{\text{sphere}} + r_{\text{capsule}}) - \text{dist}$$

### Capsule vs Capsule

두 세그먼트 사이의 최단 거리:

$$\text{dist} = \text{segmentToSegmentDistance}(\text{seg}_A, \text{seg}_B)$$

$$\text{dist} < (r_A + r_B) \quad \Rightarrow \quad \text{hit}$$

> **실전 팁**: 인간형 캐릭터는 캡슐(키 ~ 1.8m, 반지름 ~ 0.3m)로 표현하는 것이 가장 자연스럽다. AABB는 회전 시 어색하고, 구는 키가 안 맞는다.

---

## 9. 충돌 응답 (Collision Response)

충돌을 감지한 후 물리적으로 반응한다.

> **범위**: 이 문서는 충돌 **검출**이 주제이며, 이 섹션은 응답의 기초만 요약한다. 물리 응답(임펄스 · 마찰 · slop)의 상세는 《충돌처리》를 참고하라.

### 위치 보정 (Positional Correction)

관통 해결: MTV 방향으로 밀어냄.

$$\text{totalInvMass} = \frac{1}{m_A} + \frac{1}{m_B}$$

질량에 비례하여 분배:

$$\text{correction} = \mathbf{n}_{\text{mtv}} \cdot \frac{\text{depth}}{\text{totalInvMass}} \cdot 0.8$$

$$\mathbf{A}_{\text{pos}} \mathrel{-}= \text{correction} \cdot \frac{1}{m_A}$$

$$\mathbf{B}_{\text{pos}} \mathrel{+}= \text{correction} \cdot \frac{1}{m_B}$$

### 속도 반사 (Velocity Reflection)

속도를 법선 방향으로 분해 (단위 법선 $\hat{\mathbf n}$):

$$\mathbf{v}_n = (\mathbf{v} \cdot \hat{\mathbf{n}}) \, \hat{\mathbf{n}} \quad \text{(법선 성분)}$$

$$\mathbf{v}_t = \mathbf{v} - \mathbf{v}_n \quad \text{(접선 성분)}$$

반사 (반발 계수 $e$):

$$\mathbf{v}_{\text{new}} = \mathbf{v}_t - e\,\mathbf{v}_n = \mathbf{v} - (1+e)(\mathbf{v} \cdot \hat{\mathbf{n}})\,\hat{\mathbf{n}}$$

($e = 0$: 비탄성, $e = 1$: 완전 탄성)

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