---
title: 투영 행렬 (Projection Matrix)
slug: projection
---

## 소개

투영 행렬은 3D 공간의 점을 2D 화면에 그리기 위해 변환하는 행렬이다. 3D 그래픽스 파이프라인의 마지막 변환 단계로, 원근감이 있는 원근 투영과 원근감이 없는 직교 투영으로 나뉜다.

> **핵심**
> 원근 투영은 먼 것이 작게, 직교 투영은 거리 무관하게 같은 크기로 그린다.

---

## 1. 원근 투영 (Perspective Projection)

실제 카메라처럼 먼 물체가 작게 보이는 투영이다.

### 파라미터

- **FOV (Field of View)**: 시야각 (보통 수직 FOV)
- **Aspect Ratio**: 화면 종횡비 (width / height)
- **Near Plane (n)**: 가까운 절단면
- **Far Plane (f)**: 먼 절단면

### 원근 투영 행렬

$$
\mathbf{P} = \begin{bmatrix}
\frac{1}{\text{aspect} \cdot \tan(\text{fov}/2)} & 0 & 0 & 0 \\
0 & \frac{1}{\tan(\text{fov}/2)} & 0 & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2 \cdot f \cdot n}{f-n} \\
0 & 0 & -1 & 0
\end{bmatrix}
$$

### 작동 원리

```text
// 카메라 공간 점 (x, y, z, 1)을 투영
clip = P × (x, y, z, 1)
     = (x', y', z', -z)

// 원근 분할 (Perspective Divide)
ndc = (x'/(-z), y'/(-z), z'/(-z))
     = (x·focal/(-z), y·focal/(-z), ...)
```

깊이 $-z$(즉 $|z|$)가 클수록 (멀수록) $x', y'$가 작아져서 물체가 작게 보인다.

### FOV와 focal length

$$
\text{focal} = \frac{1}{\tan(\text{fov} / 2)}
$$

```text
// FOV가 클수록 focal이 작아짐 (광각, 어안렌즈 효과)
// FOV가 작을수록 focal이 커짐 (망원, 줌 효과)
```

**게임에서의 활용:**
- **1인칭/3인칭 카메라**: 일반적으로 FOV 60~90도
- **스나이퍼 줌**: FOV를 일시적으로 낮춤 (예: 90° → 20°)
- **광각 효과**: FOV를 높여서 속도감 강조

---

## 2. 직교 투영 (Orthographic Projection)

원근감 없이 평행하게 투영한다. 먼 물체나 가까운 물체나 같은 크기로 보인다.

### 파라미터

- **Left, Right (l, r)**: 수평 절단면
- **Bottom, Top (b, t)**: 수직 절단면
- **Near, Far (n, f)**: 깊이 절단면

### 직교 투영 행렬

$$
\mathbf{O} = \begin{bmatrix}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\
0 & 0 & -\frac{2}{f-n} & -\frac{f+n}{f-n} \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

### 원근 투영과의 차이

| 특징 | 원근 투영 | 직교 투영 |
|------|----------|----------|
| 원근감 | 있음 | 없음 |
| w 성분 | -z (깊이 의존) | 1 (항상 1) |
| 거리에 따른 크기 | 멀수록 작음 | 동일 |
| 용도 | 3D 게임 일반 | UI, 미니맵, 2D 게임 |

**게임에서의 활용:**
- **UI 렌더링**: 화면 좌표계로 그리기
- **미니맵**: 위에서 본 평면 투영
- **레벨 에디터**: 정면/측면/상면 뷰
- **2D 게임**: 픽셀 아트 등

---

## 3. NDC (Normalized Device Coordinates)

투영 후 모든 좌표가 $[-1, 1]$ 범위로 정규화된 공간이다.

$$
x_{\text{ndc}} \in [-1, 1] \quad \text{좌}(-1) \sim \text{우}(+1)
$$
$$
y_{\text{ndc}} \in [-1, 1] \quad \text{하}(-1) \sim \text{상}(+1)
$$
$$
z_{\text{ndc}} \in [-1, 1] \quad \text{근}(-1) \sim \text{원}(+1) \quad [\text{OpenGL}]
$$
$$
z_{\text{ndc}} \in [0, 1] \quad [\text{DirectX}]
$$

### NDC → 화면 좌표 (Viewport Transform)

$$
\text{screenX} = (x_{\text{ndc}} + 1) \times 0.5 \times \text{screenWidth}
$$
$$
\text{screenY} = (1 - y_{\text{ndc}}) \times 0.5 \times \text{screenHeight} \quad \text{(Y축 반전)}
$$

> **주의**: OpenGL은 $z \in [-1, 1]$, DirectX는 $z \in [0, 1]$을 사용한다. 엔진에 따라 투영 행렬의 z 부분이 다르다.

---

## 4. 깊이 (Depth)와 Z-버퍼

### 깊이 값의 비선형성

원근 투영 후 z값은 비선형적으로 매핑된다:

$$
z_{\text{ndc}} = \frac{A \cdot z + B}{z} \quad \text{(A, B는 near/far로 결정)}
$$

여기서 $z$는 시선 방향 깊이($= -z_{view}$, 양수)다. $A=(f+n)/(f-n),\ B=-2fn/(f-n)$로 두면 $z=n \to z_{ndc}=-1$, $z=f \to z_{ndc}=+1$로 매핑된다.

이로 인해:
- **근처 해상도 높음**: near plane 근처의 z값이 넓게 퍼짐
- **원거리 해상도 낮음**: far plane 근처의 z값이 밀집됨
- **Z-파이팅**: 먼 거리에서 깊이값이 겹쳐 깜빡임

### Z-파이팅 방지

```text
// 1. Near plane을 최대한 멀리
near = 0.1  →  1.0     // 근처 해상도 향상

// 2. Far/Near 비율 최소화
ratio = far / near      // 이 값이 클수록 z-파이팅 심함
// 권장: ratio < 1000

// 3. Reverse-Z 사용 (최신 엔진)
// 0 = far, 1 = near → 원거리 정밀도 향상
```

### Reversed-Z

```text
// 전통적(D3D [0,1]): near = 0, far = 1 (근처가 정밀) — OpenGL(-1~1)은 near = -1, far = 1
// Reversed-Z: near = 1, far = 0 (원거리가 정밀)
// 부동소수점 정밀도 분포상 Reversed-Z가 전체적으로 더 균등
```

---

## 5. 절두체 (Frustum)와 컬링

투영 행렬은 절두체(Frustum)를 정의한다. 이 영역 밖의 객체는 렌더링하지 않는다.

### 절두체 평면 추출

```text
// 투영 × 뷰 행렬 = VP 행렬
// Gribb-Hartmann 알고리즘: VP 행렬의 행 조합으로 6개 평면 추출
planes[0] = row4 + row1   // Left:   x' >= -w
planes[1] = row4 - row1   // Right:  x' <=  w
planes[2] = row4 + row2   // Bottom: y' >= -w
planes[3] = row4 - row2   // Top:    y' <=  w
planes[4] = row4 + row3   // Near:   z' >= -w (OpenGL) 또는 row3 (DirectX)
planes[5] = row4 - row3   // Far:    z' <=  w

// ★ 중요: 평면 방정식 (A, B, C, D)을 반드시 법선 길이로 정규화해야 함!
for (int i = 0; i < 6; i++) {
    float length = sqrt(planes[i].x * planes[i].x + planes[i].y * planes[i].y + planes[i].z * planes[i].z);
    planes[i] /= length; // 이제 A*x + B*y + C*z + D = 유클리드 부호 있는 거리
}
```

### 절두체 컬링 (Frustum Culling)

```text
// 객체의 바운딩 구(Sphere)가 절두체 내에 있는지 검사
function inFrustum(sphere, planes):
    for plane in planes:
        // 정규화된 평면과의 거리: dot(plane.normal, sphere.center) + plane.D
        if distanceToPlane(sphere.center, plane) < -sphere.radius:
            return false    // 바깥쪽으로 반지름 이상 벗어남 → 렌더링 제외
    return true
```

**게임에서의 활용:**
- **렌더링 최적화**: 화면 밖 객체를 그리지 않음
- **LOD 선택**: 거리에 따라 메시 디테일 조정
- **그림자 컬링**: 그림자 범위 밖 객체 제외

---

## 6. FOV와 화면 비율

### 수직 FOV vs 수평 FOV

$$
\text{horizontalFOV} = 2 \times \arctan\left(\text{aspect} \times \tan\left(\frac{\text{verticalFOV}}{2}\right)\right)
$$

```text
// 예: 16:9 화면에서 수직 FOV 60도
aspect = 16/9 = 1.778
horizontalFOV = 2 × atan(1.778 × tan(30°)) ≈ 91.5도
```

### 울트라와이드 대응

```text
// 16:9 → 21:9 모니터에서 같은 수직 FOV 유지
// 수평만 넓어짐 (올바른 방식)
// 반대로 수평 FOV를 고정하면 수직이 잘림 (잘못된 방식)
```

---

## 7. 클립 공간과 원근 분할

### 변환 파이프라인

```text
1. 로컬 좌표 (Local)
   ↓ modelMatrix
2. 월드 좌표 (World)
   ↓ viewMatrix
3. 카메라 좌표 (View/Camera)
   ↓ projectionMatrix
4. 클립 좌표 (Clip)
   ↓ ÷ w (원근 분할)
5. NDC (Normalized Device Coordinates)
   ↓ viewport transform
6. 화면 좌표 (Screen)
```

### 원근 분할의 의미

$$
\text{cw} = -z_{\text{view}} \quad \text{(카메라 공간 깊이의 음수)}
$$

$$
\text{ndc} = \left(\frac{c_x}{c_w}, \frac{c_y}{c_w}, \frac{c_z}{c_w}\right)
$$

$z$가 멀수록 $c_w$가 커지고, $x$와 $y$가 작아진다 → 원근 효과

---

## 8. 실전 예시: 동적 FOV (스프린트)

```text
// 달릴 때 FOV를 넓혀서 속도감 표현
targetFOV = isSprinting ? 90 : 70
// 프레임 독립 지수 감쇠 (《지수 감쇠》 참고): rate = 5
t = 1 - exp(-5 × dt)
currentFOV = lerp(currentFOV, targetFOV, t)
projectionMatrix = perspective(currentFOV, aspect, near, far)
```

---

## 9. 빠른 참조

| 항목 | 원근 투영 | 직교 투영 |
|------|---------|---------|
| 행렬 | 4×4, w=-z | 4×4, w=1 |
| 원근감 | 있음 | 없음 |
| NDC | [-1,1]³ | [-1,1]³ |
| 용도 | 3D 씬 | UI, 2D, 에디터 |
| FOV | 필요 | 불필요 (l/r/t/b) |
| 깊이 | 비선형 | 선형 |

| 파라미터 | 설명 | 팁 |
|----------|------|-----|
| FOV | 시야각 | 60~90도 일반적 |
| Aspect | 화면 비율 | width/height |
| Near | 근절단면 | 크게 할수록 z-파이팅 ↓ |
| Far | 원절단면 | near/far 비율 < 1000 |