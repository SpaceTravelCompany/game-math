---
title: 외적 (Cross Product)
slug: cross-product
---

## 소개

외적(Cross Product)은 두 벡터에 모두 수직인 **세 번째 벡터**를 만드는 연산이다. 3D 게임에서 법선 벡터 계산, 회전 축, 좌우 판별 등에 필수적으로 사용된다.

> **핵심**
> 외적의 결과는 두 입력 벡터 모두에 수직인 벡터다. 크기는 두 벡터가 이루는 평행사변형의 넓이다.

---

## 1. 정의

### 3D 외적

두 3D 벡터 $\mathbf{a}$, $\mathbf{b}$의 외적 $\mathbf{a} \times \mathbf{b}$는:

$$
\mathbf{a} \times \mathbf{b} = \begin{bmatrix}
a_y b_z - a_z b_y \\
a_z b_x - a_x b_z \\
a_x b_y - a_y b_x
\end{bmatrix}
$$

### 수치 계산 예시

예시:

```text
// 기저 벡터: e_x × e_y = e_z
(1,0,0) × (0,1,0) = (0·0 - 0·1, 0·0 - 1·0, 1·1 - 0·0)
                  = (0, 0, 1) = e_z

// 일반 벡터
a = (1, 2, 3),  b = (4, 5, 6)
a × b = (2×6 - 3×5, 3×4 - 1×6, 1×5 - 2×4)
      = (12-15, 12-6, 5-8)
      = (-3, 6, -3)
```

### 기하적 의미

$$
|\mathbf{a} \times \mathbf{b}| = |\mathbf{a}| \cdot |\mathbf{b}| \cdot \sin\theta
$$

- 결과 벡터의 **크기** = 두 벡터가 이루는 평행사변형의 넓이
- 결과 벡터의 **방향** = a에서 b로 오른손을 감았을 때 엄지 방향 (오른손 법칙)

### 오른손 좌표계 vs 왼손 좌표계

| 좌표계 | 엔진 | 외적 수식 (좌표계 무관 동일) |
|--------|------|------------------------------|
| 오른손 좌표계 | OpenGL, Vulkan | $\mathbf{a} \times \mathbf{b} = \mathbf{n}$ (오른손 법칙으로 해석) |
| 왼손 좌표계 | DirectX, Unity | $\mathbf{a} \times \mathbf{b} = \mathbf{n}$ (왼손 법칙으로 해석) |

> **주의**: 외적 공식 자체는 좌표계 핸드니스와 무관하게 같다. 차이는 **시각적 방향 해석**에 있다. 같은 외적 결과라도 오른손 좌표계에서는 오른손 법칙, 왼손 좌표계에서는 왼손 법칙으로 해석된다. 엔진마다 forward 벡터의 방향(+Z 또는 -Z)이 다르므로, 외적 순서나 부호가 다르게 나타날 수 있다.

---

## 2. 외적의 성질

| 성질 | 수식 | 설명 |
|------|------|------|
| 반교환법칙 | $\mathbf{a} \times \mathbf{b} = -(\mathbf{b} \times \mathbf{a})$ | 순서 바뀌면 부호 반전 |
| 영벡터 | $\mathbf{a} \times \mathbf{a} = \mathbf{0}$ | 평행한 벡터의 외적은 0 |
| 평행 | $\mathbf{a} \times \mathbf{b} = \mathbf{0} \iff \mathbf{a} \parallel \mathbf{b}$ | 평행 판별 |
| 분배법칙 | $\mathbf{a} \times (\mathbf{b} + \mathbf{c}) = \mathbf{a} \times \mathbf{b} + \mathbf{a} \times \mathbf{c}$ | 덧셈에 대해 분배 |
| 스칼라 | $(s \cdot \mathbf{a}) \times \mathbf{b} = s(\mathbf{a} \times \mathbf{b})$ | 스칼라 분리 |
| 결합법칙 부재 | $\mathbf{a} \times (\mathbf{b} \times \mathbf{c}) \ne (\mathbf{a} \times \mathbf{b}) \times \mathbf{c}$ | 외적은 결합법칙 성립 안 함 |
| 라그랑주 | $\mathbf{a} \times (\mathbf{b} \times \mathbf{c}) = \mathbf{b}(\mathbf{a} \cdot \mathbf{c}) - \mathbf{c}(\mathbf{a} \cdot \mathbf{b})$ | BAC-CAB 공식 |

---

## 3. 법선 벡터 (Normal Vector)

삼각형의 세 꼭지점으로부터 면의 법선을 계산한다:

```text
// 삼각형 ABC의 법선
edge1 = B - A
edge2 = C - A
normal = normalize(cross(edge1, edge2))
```

**게임에서의 활용:**
- **조명**: 셰이더에서 표면의 빛 반사 계산
- **충돌**: 면에 대한 충돌 법선
- **컬링**: 삼각형이 카메라를 향하는지 판별 (백페이스 컬링)

```text
// 백페이스 컬링
faceNormal = cross(B - A, C - A)
viewDir = camera.position - A

if dot(faceNormal, viewDir) < 0:
    // 삼각형이 카메라를 등지고 있음 → 렌더링 생략
```

> **와인딩 규약 주의**: `cross(B-A, C-A)`가 앞면을 가리키려면 반시계(CCW) 앞면 규약이 전제다. OpenGL 기본은 CCW, D3D 기본은 CW라 부호가 뒤집힌다(위 로직은 해당 규약下에서 정상 동작).

---

## 4. 좌우 판별 (2D 외적)

2D에서 외적은 스칼라값을 반환하며, 점이 선분의 왼쪽/오른쪽 어디에 있는지 판별한다.

### 2D 외적

$$
\mathbf{a} \times \mathbf{b} = a_x b_y - a_y b_x
$$

### 점이 선분의 어느 쪽에 있는지

```text
// 선분 AB에 대해 점 P가 왼쪽인지 오른쪽인지
AB = B - A
AP = P - A
result = cross(AB, AP)

if result > 0:  // P는 AB의 왼쪽
if result < 0:  // P는 AB의 오른쪽
if result == 0:  // P는 AB 위에 있음 (일직선)
```

**게임에서의 활용:**
- **충돌 검출**: 점이 다각형 내부에 있는지 (동일 부호(same-side)/와인딩 판별)
- **AI 내비게이션**: 경로의 왼쪽/오른쪽 판단
- **컨벡스 헐 (Convex Hull)**: 꼭짓점의 방향성 확인

---

## 5. 회전 축과 각도

외적의 방향은 회전 축이 되고, 내적으로부터 각도를 구할 수 있다:

```text
axis = normalize(cross(a, b))           // 회전 축
angle = acos(dot(normalize(a), normalize(b)))  // 회전 각도
```

> **퇴화 케이스 주의**: $\mathbf{a} \parallel \mathbf{b}$($\text{dot} \approx \pm 1$)이면 cross = 0이라 normalize가 NaN이 된다 — 평행(dot ≈ 1)이면 항등 회전으로 처리하고, 역평행(dot ≈ −1)이면 임의 수직축으로 폴백해야 한다. acos 인자도 $[-1, 1]$로 clamp한다.

**게임에서의 활용:**
- **쿼터니언 생성**: 두 방향 벡터 사이의 회전
- **카메라 회전**: 타겟을 향해 회전
- **물리**: 토크(Torque) 계산

```text
// 토크 계산
r = position - pivot       // 회전 중심에서의 상대 위치
F = appliedForce           // 가해진 힘
torque = cross(r, F)       // 회전축 + 크기
```

---

## 6. 삼중적 (Triple Product)

### 스칼라 삼중적 (평행육면체 부피)

$$
\mathbf{a} \cdot (\mathbf{b} \times \mathbf{c})
$$

$\mathbf{a} \cdot (\mathbf{b} \times \mathbf{c})$는 부호 있는 부피(기저의 handedness 반영)이며, 기하적 부피는 $V = |\mathbf{a} \cdot (\mathbf{b} \times \mathbf{c})|$다. $V = 0$(절댓값 0)이면 세 벡터는 한 평면 위에 있다 (coplanar).

### 벡터 삼중적 (BAC-CAB)

$$
\mathbf{a} \times (\mathbf{b} \times \mathbf{c}) = \mathbf{b}(\mathbf{a} \cdot \mathbf{c}) - \mathbf{c}(\mathbf{a} \cdot \mathbf{b})
$$

**게임에서의 활용:**
- **평면 방정식**: 세 점으로 평면 구하기
- **빌보드 변환**: 카메라를 향하는 평면 구성

---

## 7. 실전 예시: 카메라 Right/Up 벡터

3D 카메라에서 forward 벡터로부터 right와 up 벡터를 계산:

```text
forward = normalize(target - cameraPos)
worldUp = (0, 1, 0)

right = normalize(cross(forward, worldUp))
up = cross(right, forward)

// 이제 카메라 행렬을 구성할 수 있다
// View 행렬: 회전부 = [right, up, -forward]ᵀ, 이동부 = -Rᵀ·cameraPos (표기는 《LookAt》 §1 참고)
```

> **주의**: 여기서 forward는 '타깃을 향하는' 방향(+Z 쪽). 반면 **《LookAt & 카메라》 문서 §1**의 forward는 `eye - target`(카메라가 바라보는 -Z 방향)으로 부호가 반대이므로, 두 문서의 forward는 '향하는 방향' 정의가 다름에 유의.

---

## 8. 실전 예시: 점이 삼각형 내부인지 검사

외적을 이용해 점이 삼각형 내부에 있는지 판별한다:

```text
// 2D(z 성분만으로 판정). 3D 삼각형은 dot(d1,d2)의 부호 일치로 일반화
// 삼각형 ABC, 점 P
function pointInTriangle(A, B, C, P):
    // 각 변에 대해 같은 방향(왼쪽 또는 오른쪽)인지 확인
    d1 = cross(B - A, P - A)
    d2 = cross(C - B, P - B)
    d3 = cross(A - C, P - C)

    hasNeg = (d1 < 0) or (d2 < 0) or (d3 < 0)
    hasPos = (d1 > 0) or (d2 > 0) or (d3 > 0)

    return not (hasNeg and hasPos)  // 전부 같은 부호면 내부
```

---

## 9. 빠른 참조

| 연산 | 수식 | 활용 |
|------|------|------|
| 3D 외적 | $\mathbf{a} \times \mathbf{b}$ | 법선, 회전축 |
| 2D 외적 | $a_x \cdot b_y - a_y \cdot b_x$ | 좌우 판별 |
| 크기 | $|\mathbf{a} \times \mathbf{b}| = |\mathbf{a}||\mathbf{b}|\sin\theta$ | 평행사변형 넓이 |
| 평행 판별 | $\mathbf{a} \times \mathbf{b} = \mathbf{0}$ | 평행 확인 |
| 법선 | $\text{cross}(\mathbf{B}-\mathbf{A}, \mathbf{C}-\mathbf{A})$ | 삼각형 법선 |
| 부피 | $|\mathbf{a} \cdot (\mathbf{b} \times \mathbf{c})|$ | 평행육면체 기하 부피 (내부 스칼라 삼중적은 부호 있음) |
| 회전축 | $\text{normalize}(\text{cross}(\mathbf{a}, \mathbf{b}))$ | a→b 회전 |