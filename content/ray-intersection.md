---
title: 광선 교차 (Ray Intersection)
slug: ray-intersection
---

## 소개

광선 교차(Ray Intersection)는 광선(Ray)이 기하학적 형태와 만나는지 검사하는 알고리즘이다. 마우스 픽킹, 총알 판정, 시야 확인, 그림자 레이캐스팅, 광선 추적 렌더링 등 게임 전반에 걸쳐 사용된다.

> **핵심**
> 광선 = 원점 + 방향 × t. t를 구하면 교차점을 알 수 있다.

---

## 1. 광선의 정의

$$R(t) = \mathbf{O} + t \cdot \mathbf{D}$$

- $\mathbf{O}$: 원점 (origin)
- $\mathbf{D}$: 방향 (단위 벡터)
- $t$: 매개변수 ($t \ge 0$)

- $t > 0$: 원점에서 앞쪽 방향의 점
- $t = 0$: 원점 자체
- $t < 0$: 원점 뒤쪽 (보통 무시)

---

## 2. Ray-Plane 교차

### 평면의 정의

점 $\mathbf{P_0}$과 법선 $\mathbf{n}$으로 정의된 평면:

$$\mathbf{n} \cdot (\mathbf{P} - \mathbf{P_0}) = 0$$

### 교차 계산

$R(t) = \mathbf{O} + t\mathbf{D}$ 가 평면 위에 있을 때:

$$\mathbf{n} \cdot ((\mathbf{O} + t\mathbf{D}) - \mathbf{P_0}) = 0$$

$t$에 대해 정리:

$$t = \frac{\mathbf{n} \cdot (\mathbf{P_0} - \mathbf{O})}{\mathbf{n} \cdot \mathbf{D}}$$

- 분모가 0이면 광선과 평면이 평행 (교차 없음): $\mathbf{n} \cdot \mathbf{D} = 0 \Rightarrow$ no hit
- $t < 0$이면 교차점이 광선 뒤: no hit

교차점:

$$\mathbf{P}_{\text{hit}} = \mathbf{O} + t \cdot \mathbf{D}$$

**게임에서의 활용:**
- **마우스 클릭**: 화면 좌표 → 월드 광선 → 지면(평면)과 교차
- **지면 스냅**: 객체를 지형 위에 배치
- **평면 반사**: 거울, 물면 반사

---

## 3. Ray-Sphere 교차

### 구의 정의

중심 $\mathbf{C}$, 반지름 $r$인 구:

$$\|\mathbf{P} - \mathbf{C}\| = r$$

### 교차 계산 (해석적 방법)

$R(t) = \mathbf{O} + t\mathbf{D}$를 구 방정식에 대입:

$$\|\mathbf{O} + t\mathbf{D} - \mathbf{C}\|^2 = r^2$$

$\mathbf{f} = \mathbf{O} - \mathbf{C}$라 하면:

$$a = \mathbf{D} \cdot \mathbf{D} \quad ({}= 1 \text{, 단위 벡터인 경우})$$

$$b = 2 \, \mathbf{f} \cdot \mathbf{D}$$

$$c = \mathbf{f} \cdot \mathbf{f} - r^2$$

판별식:

$$\Delta = b^2 - 4ac$$

- $\Delta < 0$: 교차 없음
- $\Delta = 0$: 접선(tangent) — 한 점에서 스침, $t_1 = t_2 = \dfrac{-b}{2a}$
- $\Delta > 0$: 두 근 (아래)

두 근:

$$t_1 = \frac{-b - \sqrt{\Delta}}{2a}, \quad t_2 = \frac{-b + \sqrt{\Delta}}{2a}$$

가장 가까운 양수 $t$:

$$t = \begin{cases} t_1 & \text{if } t_1 \ge 0 \\ t_2 & \text{if } t_2 \ge 0 \\ \text{no hit} & \text{otherwise} \end{cases}$$

### 기하적 방법 (더 직관적)

광선 원점에서 구 중심까지의 벡터:

$$\mathbf{L} = \mathbf{C} - \mathbf{O}$$

광선에 대한 투영 거리:

$$t_c = \mathbf{L} \cdot \mathbf{D}$$

- $t_c < 0$ **이고 원점이 구 밖이면** ($\|\mathbf{L}\| > r$): 구가 광선 뒤에 있음 → no hit (원점이 구 내부인 경우는 아래 주의 참고)

투영점에서 구 중심까지의 거리:

$$d^2 = \mathbf{L} \cdot \mathbf{L} - t_c^2$$

- $d^2 > r^2$: 광선이 구를 스치지 않음 → no hit
- $d^2 = r^2$: 접선(tangent) — 한 점에서 스침 ($t_l = 0$, $t_1 = t_2 = t_c$)

교차점까지의 거리:

$$t_l = \sqrt{r^2 - d^2}$$

$$t_1 = t_c - t_l \quad \text{(가까운 교차점)}, \quad t_2 = t_c + t_l \quad \text{(먼 교차점)}$$

> **주의 — 원점이 구 내부인 경우**
> 광선 원점이 구 내부($\|\mathbf{L}\| < r$)라면 $t_c < 0$이어도 no hit이 아니다. 이때는 항상 $t_1 < 0 < t_2$이므로, 중심이 원점 뒤쪽에 있어도($t_c < 0$) 앞쪽 출구점 $t_2 = t_c + t_l > 0$이 존재해 히트로 잡아야 한다. 해석적 방법의 "가장 가까운 양수 $t$" 분기는 이 경우를 자동으로 처리한다.

**게임에서의 활용:**
- **총알 판정**: 광선이 캐릭터의 바운딩 구를 관통하는지
- **선택**: 클릭한 객체가 구 형태인 경우
- **불릿 헬**: 탄환이 구체 적을 맞추는 판정

---

## 4. Ray-AABB 교차 (Slab Method)

AABB(Axis-Aligned Bounding Box)는 축에 정렬된 박스다.

### AABB 정의

$$\text{AABB} = \{ \mathbf{P} \mid \min_x \le P_x \le \max_x,\; \min_y \le P_y \le \max_y,\; \min_z \le P_z \le \max_z \}$$

### Slab Method

각 축별로 광선이 슬랩(slab, 두 평면 사이 공간)을 통과하는 구간을 구하고, 교집합이 있는지 확인한다.

```text
function rayAABB(O, D, min, max):
    tmin = -∞
    tmax = +∞

    for each axis i in (x, y, z):
        if D[i] == 0:
            // 광선이 축에 수직 → 원점이 슬랩 안에 있어야 함
            if O[i] < min[i] or O[i] > max[i]:
                return no hit
        else:
            t1 = (min[i] - O[i]) / D[i]
            t2 = (max[i] - O[i]) / D[i]

            if t1 > t2: swap(t1, t2)

            tmin = max(tmin, t1)
            tmax = min(tmax, t2)

            if tmin > tmax:
                return no hit        // 교차 없음

    if tmax < 0:
        return no hit                // 박스가 광선 뒤에 있음

    t = (tmin ≥ 0) ? tmin : tmax
    return O + t × D
```

### 최적화 (조건문 줄이기)

```text
// 분기 없는 버전 (현대 CPU에서 더 빠름)
invD = 1 / D
t1 = (min - O) × invD
t2 = (max - O) × invD

tmin = max(min(t1.x, t2.x), min(t1.y, t2.y), min(t1.z, t2.z))
tmax = min(max(t1.x, t2.x), max(t1.y, t2.y), max(t1.z, t2.z))

hit = tmax ≥ max(tmin, 0)
```

**게임에서의 활용:**
- **마우스 픽킹**: 화면에서 광선을 쏘아 객체(AABB) 선택
- **가시성 검사**: 광선이 장애물을 관통하는지
- **물리 엔진**: 브로드페이즈 충돌 후 정밀 검사

---

## 5. Ray-Triangle 교차 (Möller–Trumbore)

삼각형은 3D 그래픽스의 기본 단위이므로 광선-삼각형 교차는 가장 중요한 교차 테스트 중 하나다.

### Möller–Trumbore 알고리즘

```text
function rayTriangle(O, D, V0, V1, V2):
    edge1 = V1 - V0
    edge2 = V2 - V0

    h = cross(D, edge2)
    a = dot(edge1, h)

    if a > -ε and a < ε:
        return no hit            // 광선이 삼각형에 평행

    f = 1 / a
    s = O - V0
    u = f × dot(s, h)

    if u < 0 or u > 1:
        return no hit            // 삼각형 바깥

    q = cross(s, edge1)
    v = f × dot(D, q)

    if v < 0 or u + v > 1:
        return no hit            // 삼각형 바깥

    t = f × dot(edge2, q)

    if t > ε:
        return hit(O + t × D, t, u, v)    // u, v: 속성 보간/픽싱용
    else:
        return no hit            // 교차점이 광선 뒤
```

### u, v의 의미

삼각형 내부를 $(u, v)$ 매개변수로 표현:

$$\mathbf{P} = (1-u-v)\mathbf{V_0} + u\mathbf{V_1} + v\mathbf{V_2}$$

$u \ge 0,\; v \ge 0,\; u+v \le 1$이면 삼각형 내부.

### 법선 계산

교차점의 법선은 꼭지점 법선의 무게 중심 가중치로 보간한다:

$$w = 1 - u - v$$

$$\mathbf{n}_{\text{hit}} = w \, \mathbf{N_0} + u \, \mathbf{N_1} + v \, \mathbf{N_2}$$

$\mathbf{N_0}, \mathbf{N_1}, \mathbf{N_2}$는 각 꼭지점의 법선.

**게임에서의 활용:**
- **정밀 충돌**: AABB 통과 후 삼각형 레벨 검사
- **광선 추적 (RTX)**: 픽셀당 광선-삼각형 교차
- **지형 따라가기**: 캐릭터가 지형 메시 위에 서 있는지
- **길찾기**: 장애물 경계와 광선 교차

---

## 6. Ray-Disc / Ray-Cylinder 교차

### Ray-Disc (원판)

평면과 교차 후 원 내부인지 확인:

$$t = \text{rayPlane}(\mathbf{O}, \mathbf{D}, \mathbf{C}_{\text{disc}}, \mathbf{n}_{\text{disc}})$$

$$\mathbf{P} = \mathbf{O} + t \cdot \mathbf{D}$$

$$\|\mathbf{P} - \mathbf{C}_{\text{disc}}\| \le r \quad \Rightarrow \quad \text{hit}$$

### Ray-Cylinder (무한 원기둥)

축 위 점 $\mathbf{Q}$, 단위 축 $\hat{\mathbf{A}}$, 반지름 $r$인 원기둥. 벡터 $\mathbf{X}$의 축 수직 성분은:

$$\mathbf{X}_\perp = \mathbf{X} - (\mathbf{X} \cdot \hat{\mathbf{A}})\,\hat{\mathbf{A}}$$

$\mathbf{m} = \mathbf{O} - \mathbf{Q}$라 하면, "축까지의 거리 $= r$" 조건에 광선을 대입해 2차 방정식을 얻는다 (Scratchapixel):

$$\|\mathbf{D}_\perp\|^2\,t^2 + 2(\mathbf{m}_\perp \cdot \mathbf{D}_\perp)\,t + (\|\mathbf{m}_\perp\|^2 - r^2) = 0$$

두 근 중 가장 작은 음이 아닌 $t$가 교차점 (판별식 $< 0$이면 no hit). 유한 원기둥은 두 캡(원판)을 추가로 검사하고, 히트점이 축의 양 끝면 사이에 있는지 제한해야 한다.

---

## 7. 광선 변환

월드 공간의 광선을 객체의 로컬 공간으로 변환하여 로컬 기하학으로 교차 검사를 수행하는 것이 더 간단할 때가 많다.

```text
// 월드 → 로컬
invModel = inverse(modelMatrix)
localOrigin = invModel × vec4(O, 1)
localDir = invModel × vec4(D, 0)    // 방향은 w=0

// 로컬 공간에서 교차 검사 (보통 단순한 기하학)
hit = raySphere(localOrigin, localDir, localCenter, radius)

// 교차점을 월드로 되돌림
worldHit = modelMatrix × vec4(localHit, 1)
```

> **주의**: 비균일 스케일이 포함된 경우 방향 벡터를 정규화해야 한다. 비균일 스케일이 있으면 로컬 구가 월드에서 타원체가 되므로, `raySphere` 검사는 균일 스케일(또는 스케일 없는 객체)에서만 정확하다.

---

## 8. 실전 예시: 마우스 픽킹

```text
// 화면 좌표 → 월드 광선
mouseX, mouseY     // 픽셀 좌표

// NDC 변환
ndcX = (2 × mouseX / screenWidth) - 1
ndcY = 1 - (2 × mouseY / screenHeight)  // Y축 반전

// 역투영: 근평면과 원평면 점
nearPoint = inverse(projection × view) × vec4(ndcX, ndcY, -1, 1)
farPoint = inverse(projection × view) × vec4(ndcX, ndcY, 1, 1)

// 원근 분할
nearPoint /= nearPoint.w
farPoint /= farPoint.w

// 광선 생성
rayOrigin = nearPoint
rayDir = normalize(farPoint - nearPoint)

// 객체들과 교차 검사
for object in scene:
    if rayAABB(rayOrigin, rayDir, object.min, object.max):
        // 정밀 검사 (삼각형 등)
        ...
```

---

## 9. 최적화 기법

### 계층적 검사

```text
1. 바운딩 구 (Sphere) — 빠른 거부 (가장 저렴)
2. AABB — 정밀 거부
3. 삼각형 — 정확한 교차 (가장 비싼 비용)
```

### 조기 종료 (Early Exit)

```text
// 첫 번째 히트만 필요한 경우 (총알 판정)
// tmax를 현재 최소 t로 제한
closestT = ∞
for obj in sortedByDistance:
    if rayAABB(O, D, obj.min, obj.max, tmin=0, tmax=closestT):
        t = rayTriangle(...)
        if t < closestT:
            closestT = t
            hitObj = obj
```

### 공간 분할 (Spatial Acceleration)

```text
// BVH (Bounding Volume Hierarchy)
// 각 노드에서 AABB 테스트 후, 히트한 자식만 재귀

// 그리드 / 옥트리
// 공간을 셀로 나누어 광선이 지나는 셀만 검사
```

---

## 10. 빠른 참조

| 교차 | 복잡도 | 활용 |
|------|--------|------|
| Ray-Plane | O(1) | 지면, 벽 |
| Ray-Sphere | O(1) | 바운딩 구, 총알 |
| Ray-AABB | O(1) | 브로드페이즈 |
| Ray-Triangle | O(1) | 정밀 충돌, RT |
| Ray-Disc | O(1) | 원형 영역 |
| Ray-Cylinder | O(1) | 캐릭터 충돌 |

| 최적화 | 효과 |
|--------|------|
| Sphere → AABB → Triangle | 조기 거부 |
| 역행렬로 로컬 변환 | 기하학 단순화 |
| BVH/Octree | N → log N |
| 조기 종료 | 첫 히트만 |