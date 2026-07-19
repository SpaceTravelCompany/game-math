---
title: 거리 & 최단점 (Distance & Closest Points)
slug: distance-points
---

## 소개

두 기하학적 객체 사이의 거리와 최단점을 구하는 것은 충돌 검출, 경로 계획, 물리 시뮬레이션, 렌더링 최적화 등에 필수적이다.

> **핵심**
> 거리² 를 먼저 계산하고, 실제 거리가 필요할 때만 sqrt를 호출하자.

---

## 1. 점-점 거리

$$d = |\mathbf{A} - \mathbf{B}| = \sqrt{(A_x - B_x)^2 + (A_y - B_y)^2 + (A_z - B_z)^2}$$

> **최적화: 제곱 거리**

$$d_{\text{Sq}} = (\mathbf{A} - \mathbf{B}) \cdot (\mathbf{A} - \mathbf{B})$$

> **실전 팁**: 비교만 필요한 경우 sqrt를 생략하고 dSq를 사용한다. 게임 루프에서 수천 번 호출되는 거리 비교에서 큰 성능 차이.

---

## 2. 점-선분 거리

점 P에서 선분 AB까지의 최단 거리.

### 공식

```text
function pointToSegment(P, A, B):
    AB = B - A
    AP = P - A

    t = dot(AP, AB) / dot(AB, AB)
    t = clamp(t, 0, 1)        // 선분 내로 제한

    closest = A + t × AB      // 선분 위의 최근접점
    d = P - closest
    return length(d), closest
```

### t값의 의미

- $t < 0$: 최근접점은 A (선분 시작점 이전)
- $t = 0$: 최근접점은 A
- $0 < t < 1$: 선분 위에 있음
- $t = 1$: 최근접점은 B
- $t > 1$: 최근접점은 B (선분 끝점 이후)

**게임에서의 활용:**
- **캡슐 충돌**: 구의 중심에서 캡슐의 세그먼트까지 거리
- **레이저 판정**: 점이 레이저 빔(선분) 근처에 있는지
- **AI**: 경로(폴리라인)에서 가장 가까운 세그먼트 찾기

---

## 3. 점-평면 거리

> 평면: 점 $\mathbf{P}_0$과 **단위** 법선 $\mathbf{n}$으로 정의 (비단위 법선이면 $d$가 법선 길이만큼 스케일됨)

$$d = \mathbf{n} \cdot (\mathbf{P} - \mathbf{P}_0)$$

- $d > 0$: P는 법선 방향 쪽
- $d < 0$: P는 법선 반대 방향 쪽
- $d = 0$: P는 평면 위

### 평면에 투영

$$\mathbf{P}_{\text{projected}} = \mathbf{P} - d \times \mathbf{n}$$

> $d$는 부호 있는 거리

**게임에서의 활용:**
- **지면 스냅**: 객체를 지형(평면) 위에 배치
- **시야 컬링**: 점이 카메라 앞/뒤인지
- **평면 반사**: 물면에 반사되는 위치 계산

---

## 4. 점-무한선 거리

선분이 아닌 무한히 연장하는 직선인 경우.

### 공식 (외적 이용)

> 직선 위의 점 $\mathbf{A}$, 방향 $\mathbf{D}$ (단위 벡터)

$$\mathbf{AP} = \mathbf{P} - \mathbf{A}$$

$$\mathbf{C}_{\text{closest}} = \mathbf{A} + (\mathbf{AP} \cdot \mathbf{D}) \times \mathbf{D}$$

$$\mathbf{d} = \mathbf{P} - \mathbf{C}_{\text{closest}}$$

$$\text{distance} = |\mathbf{d}|$$

> 또는 외적으로 (더 직관적):

$$\text{distance} = |\mathbf{AP} \times \mathbf{D}| \quad \text{(D가 단위 벡터일 때)}$$

---

## 5. 선분-선분 최단 거리

두 선분이 3D 공간에서 꼬인(skew) 경우의 최단 거리.

### 공식

```text
function segmentToSegment(A1, A2, B1, B2):
    u = A2 - A1    // 선분 A의 방향
    v = B2 - B1    // 선분 B의 방향
    w = A1 - B1

    a = dot(u, u)
    b = dot(u, v)
    c = dot(v, v)
    d = dot(u, w)
    e = dot(v, w)

    denom = a×c - b×b

    if denom < ε:
        // 평행한 경우
        // 한 점에서 다른 선분까지의 거리로 계산
        ...
    else:
        s = clamp((b×e - c×d) / denom, 0, 1)
        t = clamp((b×s + e) / c, 0, 1)
        s = clamp((b×t - d) / a, 0, 1)    // 반복 보정

    closestA = A1 + s × u
    closestB = B1 + t × v
    distance = length(closestA - closestB)
    return distance, closestA, closestB
```

**게임에서의 활용:**
- **캡슐-캡슐 충돌**: 두 캡슐의 세그먼트 거리
- **전선/케이블**: 두 전선이 너무 가까운지 검사
- **IK (역운동학)**: 관절 한계 내에서 뼈의 교차 방지

---

## 6. 점-삼각형 거리

점 P에서 삼각형 ABC까지의 최단 거리.

### 방법

```text
function pointToTriangle(P, A, B, C):
    // 1. 삼각형의 평면에 투영
    normal = normalize(cross(B-A, C-A))
    d = dot(normal, P - A)
    projected = P - d × normal

    // 2. 투영점이 삼각형 내부인지 확인 (Barycentric)
    // P = A + u×(B-A) + v×(C-A)
    v0 = B - A
    v1 = C - A
    v2 = projected - A

    d00 = dot(v0, v0)
    d01 = dot(v0, v1)
    d11 = dot(v1, v1)
    d20 = dot(v2, v0)
    d21 = dot(v2, v1)

    denom = d00 × d11 - d01 × d01
    v = (d11 × d20 - d01 × d21) / denom
    w = (d00 × d21 - d01 × d20) / denom
    u = 1 - v - w

    // 3. 내부면 투영점 거리, 외부면 가장 가까운 변/꼭지점 거리
    if u ≥ 0 and v ≥ 0 and w ≥ 0:
        return abs(d), projected    // 내부 → 평면 거리
    else:
        // 외부 → 세 변에 대해 각각 점-선분 거리 계산
        dAB = pointToSegment(P, A, B)
        dBC = pointToSegment(P, B, C)
        dCA = pointToSegment(P, C, A)
        return min(dAB, dBC, dCA)
```

**게임에서의 활용:**
- **지형 충돌**: 캐릭터 발이 지형 메시 위에 있는지
- **정밀 피킹**: 마우스가 삼각형 표면에 얼마나 가까운지
- **물리**: 객체가 삼각형 표면을 관통하는지

---

## 7. 점-AABB 거리

```text
function pointToAABB(P, box):
    closestX = clamp(P.x, box.min.x, box.max.x)
    closestY = clamp(P.y, box.min.y, box.max.y)
    closestZ = clamp(P.z, box.min.z, box.max.z)

    d = P - (closestX, closestY, closestZ)
    return length(d), (closestX, closestY, closestZ)
```

### 내부 판정

```text
// 점이 AABB 내부에 있으면 거리 = 0
if P.x ≥ box.min.x and P.x ≤ box.max.x and
   P.y ≥ box.min.y and P.y ≤ box.max.y and
   P.z ≥ box.min.z and P.z ≤ box.max.z:
    return 0    // 내부
```

### 점-OBB 거리

회전 박스(OBB)는 점을 OBB 로컬 좌표(중심 원점, 세 축이 기저)로 변환한 뒤 점-AABB처럼 clamp한다 — 점-AABB의 일반화다 (Ericson, *Real-Time Collision Detection* §5.1.4; OBB 정의는 《충돌 검출》 §4).

```text
function pointToOBB(P, obb):
    // 1. 점을 OBB 로컬 좌표로 변환 (축 = [r, u, f])
    d = P - obb.center
    px = dot(d, obb.r)
    py = dot(d, obb.u)
    pz = dot(d, obb.f)

    // 2. 로컬 좌표에서 ±halfSize로 clamp (점-AABB)
    cx = clamp(px, -obb.halfSize.x, obb.halfSize.x)
    cy = clamp(py, -obb.halfSize.y, obb.halfSize.y)
    cz = clamp(pz, -obb.halfSize.z, obb.halfSize.z)

    // 3. 월드 좌표로 되돌림
    closest = obb.center + cx × obb.r + cy × obb.u + cz × obb.f
    return length(P - closest), closest
```

---

## 8. 점-구 거리

$$d = |\mathbf{P} - \mathbf{C}_{\text{center}}| - r$$

- $d > 0$: 구 밖
- $d = 0$: 구 표면
- $d < 0$: 구 내부

---

## 9. 최근접점 활용 예시

### 벽 미끄러짐 (Wall Sliding)

```text
// 캐릭터가 벽에 충돌했을 때 벽을 따라 미끄러지기
hit = collide(character.position + velocity × dt)

if hit:
    // 법선 방향의 속도 제거, 접선 방향 유지
    velocity = velocity - dot(velocity, hit.normal) × hit.normal
    // 남은 속도로 이동
    character.position += velocity × dt
```

### 자기장/중력 영역

```text
// 선분(전선)에서 점까지의 거리로 자기장 세기 계산
dist, closest = pointToSegment(particle.position, wire.A, wire.B)
strength = charge / (dist² + softening²)
direction = normalize(closest - particle.position)
force = strength × direction
```

---

## 10. 빠른 참조

| 거리 | 공식 | 활용 |
|------|------|------|
| 점-점 | \|A-B\| | 비교, 정렬 |
| 점-선분 | clamp(dot(AP,AB)/dot(AB,AB)) | 캡슐, 경로 |
| 점-평면 | dot(n, P-P0) | 지면, 시야 |
| 점-직선 | cross(AP,D) | 회전, 광선 |
| 선분-선분 | 최근접 매개변수(2×2 선형 연립 + clamp) | 캡슐-캡슐 |
| 점-삼각형 | 바리센트릭 | 지형 충돌 |
| 점-AABB | clamp | 바운딩 박스 |
| 점-OBB | clamp(로컬 좌표) | 회전 박스 |
| 점-구 | \|P-C\|-r | 영역 검사 |

| 최적화 팁 | 설명 |
|-----------|------|
| 거리² 사용 | sqrt 생략 |
| 조기 종료 | 비교 시 첫 조건에서 거부 |
| 공간 분할 | 그리드/BVH로 후보 축소 |
| 근사 형태 | 구/AABB로 먼저 거르기 |