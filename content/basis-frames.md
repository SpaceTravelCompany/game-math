---
title: 기저·좌표계 (Basis & Coordinate Frames)
slug: basis-frames
---

## 소개

기저(basis)는 벡터 공간을 span하는 선형독립 벡터 집합이다. 좌표계는 원점(origin)과 기저의 조합으로, 공간 내 모든 점과 방향을 숫자로 표현할 수 있게 한다. 벡터와 행렬의 기초 개념이며, 게임 엔진의 모든 변환—월드, 로컬, 카메라, UV 공간—이 기저 위에서 작동한다.

> **핵심**
> 기저는 벡터 공간의 "단위 자" 역할을 한다. 어떤 벡터든 기저 벡터들의 선형결합으로 유일하게 표현할 수 있다.

---

## 1. 기저의 정의

유한 차원 벡터 공간에서 기저 $\mathcal{B} = \{\mathbf{e}_1, \dots, \mathbf{e}_n\}$는 두 조건을 만족한다:

1. **선형독립**: $c_1\mathbf{e}_1 + \cdots + c_n\mathbf{e}_n = \mathbf{0} \iff c_1 = \cdots = c_n = 0$
2. **생성(Span)**: 공간의 모든 벡터를 이들의 선형결합으로 표현 가능

```text
    ℝ²: e₁=(1,0), e₂=(0,1) → v = (3,2) = 3·e₁ + 2·e₂

    y↑
    2┊   · v (3,2)
    1┊  ╱
    0└-+--→ x
     0 1 2 3
```

### 표준 기저 (Standard Basis)

| 차원 | 표준 기저 | 설명 |
|------|-----------|------|
| 2D | $\mathbf{e}_x = (1,0),\; \mathbf{e}_y = (0,1)$ | x, y축 방향 |
| 3D | $\mathbf{e}_x = (1,0,0),\; \mathbf{e}_y = (0,1,0),\; \mathbf{e}_z = (0,0,1)$ | x, y, z축 방향 |

### 임의의 기저

표준 기저가 아닌 다른 기저도 얼마든지 가능하다. 단, 기저 벡터들은 반드시 서로 선형독립이어야 한다:

```text
    비표준 기저 예시     임의 벡터 (3,2)의 표현
    e₁=(2,0)  e₂=(1,2)     (3,2) = 1.0·e₁ + 1.0·e₂
                           = (2, 0) + (1, 2)
                           = (3, 2)  ← B-좌표 (1.0, 1.0)
```

> 같은 점이라도 **기저가 다르면 좌표 표현이 달라진다**. 이것이 좌표계 변환의 핵심이다.

---

## 2. 좌표 변환 (Change of Basis)

한 기저 $\mathcal{B}$로 표현된 좌표를 다른 기저 $\mathcal{B}'$로 변환하는 방법.

### 변환 행렬

기저 $\mathcal{B} = \{\mathbf{e}_1, \dots, \mathbf{e}_n\}$에서 표준 기저로의 변환 행렬은 기저 벡터들을 **열**로 쌓으면 된다:

$$
M_{\mathcal{B}} = \begin{bmatrix}
\mathbf{e}_1 & \mathbf{e}_2 & \cdots & \mathbf{e}_n
\end{bmatrix}
$$

표준 기저 좌표 $\mathbf{v}_{\text{std}}$와 $\mathcal{B}$-좌표 $\mathbf{v}_{\mathcal{B}}$의 관계:

$$
\mathbf{v}_{\text{std}} = M_{\mathcal{B}} \, \mathbf{v}_{\mathcal{B}}
$$

```text
// 기저 B 좌표 (a, b) → 표준 좌표
v_std = a × e1 + b × e2
```

### 기저 간 변환

기저 $\mathcal{B}$에서 기저 $\mathcal{B}'$로의 변환:

$$
\mathbf{v}_{\mathcal{B}'} = M_{\mathcal{B}'}^{-1} \, M_{\mathcal{B}} \, \mathbf{v}_{\mathcal{B}}
$$

```text
    기저 B 좌표 → [M_B] → 표준 좌표 → [M_B'⁻¹] → 기저 B' 좌표
```

> **게임 활용**: 로컬 공간(모델 기저)의 정점을 월드 공간(월드 기저)으로 옮길 때 사용한다. 《변환 행렬》 §5(좌표계 변환) 참고.

> **점과 방향 벡터**: 점은 원점 이동(평행이동)의 영향을 받아 프레임 이동분이 함께 더해지고, 방향 벡터는 평행이동을 무시하고 $3\times3$ 선형 부분만 적용한다.

---

## 3. 정규 직교 기저 (Orthonormal Basis)

기저 벡터들이 서로 수직이고(직교) 각각 길이가 1인(정규) 특별한 기저:

$$
\mathbf{e}_i \cdot \mathbf{e}_j = \delta_{ij} =
\begin{cases}
1 & i = j \\
0 & i \ne j
\end{cases}
$$

```text
    정규 직교         비직교
    e₂↑              e₂↗
      |                | /
      +--→ e₁          +/→ e₁
    e₁·e₂=0           e₁·e₂≠0
    |e₁|=|e₂|=1
```

### 정규 직교 기저의 장점

| 성질 | 수식 | 이점 |
|------|------|------|
| 역행렬 = 전치 | $M^{-1} = M^{\mathsf{T}}$ | 계산 비용 대폭 감소 |
| 내적 보존 | $M\mathbf{a} \cdot M\mathbf{b} = \mathbf{a} \cdot \mathbf{b}$ | 거리와 각도 유지 |
| 노름 보존 | $\|M\mathbf{v}\| = \|\mathbf{v}\|$ | 길이가 변하지 않음 |

> **Handedness**: 정규 직교 기저 행렬의 행렬식이 $\det = +1$이면 회전(같은 handedness), $-1$이면 반사(handedness 반전)다. 《외적》 §1 참고.

> **회전 행렬**의 열(또는 행)은 항상 정규 직교 기저를 이룬다. 《행렬 기초》 §7(직교 행렬)과 《변환 행렬》 §3(회전 행렬) 참고.

```text
// 3D 회전 행렬 R의 열 = 정규 직교 기저
R = [ right | up | forward ]
// Rᵀ × R = I, det(R) = 1
```

---

## 4. 좌표계 (Frame)

좌표계는 **원점(Origin)**과 **기저(Basis)**의 조합이다. 좌표계 간 이동 시 점과 방향 벡터의 처리 차이는 §2를 참고한다.

```text
    좌표계 F = { O, e₁, e₂, e₃ }

    O = 원점 위치
    e₁, e₂, e₃ = 기저 벡터들
```

### 주요 좌표계

| 좌표계 | 원점 | 기저 | 용도 |
|--------|------|------|------|
| **월드(World)** | 게임 세계의 기준점 $(0,0,0)$ | 표준 기저 $(X,Y,Z)$ | 모든 객체의 절대 위치 |
| **로컬(Local)** | 객체의 중심 | 객체의 right/up/forward | 모델 정점 정의 |
| **카메라(View)** | 카메라 위치 | right/up/-forward | 렌더링 |
| **UV** | $(0,0)$ (텍스처 좌하단 — OpenGL 관례; D3D·Vulkan은 좌상단) | $(1,0), (0,1)$ | 텍스처 매핑 |
| **스크린** | 좌상단 또는 좌하단 | 픽셀 단위 | 2D UI/출력 |

```text
              월드 y↑
                   |   로컬 (객체)
                   |   forward↑
                   |       |
                   |       +--→ right
                   +----------→ x
                  /
                 ↓ z

    로컬 → 월드 = 객체의 변환 행렬(TRS) 적용
```

월드 좌표계와 로컬 좌표계 사이의 변환은 《변환 행렬》 §5, 카메라 좌표계는 《LookAt & 카메라》 문서에서 자세히 다룬다.

---

## 5. 응용

### TBN (Tangent Basis) — 노멀 매핑

노멀 매핑은 각 픽셀의 법선을 텍스처(노멀맵)에서 읽어와 **TBN 기저**를 통해 월드 공간으로 변환한다:

```text
// TBN = Tangent, Bitangent, Normal 기저
// 노멀맵의 (R,G,B) = tangent-space 법선

T = normalize(tangent)              // UV의 U 방향
B = normalize(bitangent)            // UV의 V 방향 (= cross(N, T))
N = normalize(normal)               // 기하 법선

// TBN 행렬: tangent-space → world-space
TBN = [ T_x B_x N_x ]
      [ T_y B_y N_y ]
      [ T_z B_z N_z ]

// 노멀맵 샘플 → 월드 법선
normal_world = TBN × normal_map.rgb
```

TBN의 B(bitangent)는 《외적》으로 구한다. 자세한 내용은 《외적》 §3(법선 벡터) 참고.

### 빌보드 (Billboard)

빌보드는 항상 카메라를 향하는 2D 스프라이트다. 카메라의 right/up을 기저로 사각형을 구성한다:

```text
// 카메라 facing 기저로 빌보드 사각형 구성
halfW, halfH = 0.5 * width, 0.5 * height
R, U = camera.right, camera.up
center = position

v0 = center - R*halfW - U*halfH
v1 = center + R*halfW - U*halfH
v2 = center + R*halfW + U*halfH
v3 = center - R*halfW + U*halfH
```

### 객체 배치 (Instance Placement)

여러 객체를 장면에 배치할 때 각 객체는 자신만의 로컬 좌표계(기저)를 가진다. TRS 행렬이 이 기저를 인코딩한다:

```text
// 나무 3개를 다양한 위치/회전으로 배치
tree1.matrix = TRS(pos=(0,0,0), rot=0°, scale=1)
tree2.matrix = TRS(pos=(5,0,3), rot=45°, scale=1.2)
tree3.matrix = TRS(pos=(2,0,-4), rot=-30°, scale=0.9)
```

각 행렬의 $3\times3$ 상단 블록이 해당 객체의 로컬 기저(right/up/forward)다. 단, scale ≠ 1이면 열은 **스케일이 곱해진** 기저 벡터다 — 단위 벡터가 필요하면 각 열을 정규화하고, scale = 1이면 열이 그대로 (단위) 기저 벡터다.

---

## 6. 빠른 참조

| 개념 | 정의 | 핵심 |
|------|------|------|
| 기저 (Basis) | 선형독립인 벡터 집합 | 모든 벡터를 유일하게 표현 |
| 표준 기저 | $(1,0),(0,1)$ (2D) / $(1,0,0),(0,1,0),(0,0,1)$ (3D) | 가장 단순한 기저 |
| 정규 직교 기저 | $\mathbf{e}_i \cdot \mathbf{e}_j = \delta_{ij}$ | 역행렬 = 전치 |
| 좌표계 (Frame) | 원점 + 기저 | 월드/로컬/카메라/UV |
| 변환 행렬 | $M_{\mathcal{B}} = [\mathbf{e}_1 \cdots \mathbf{e}_n]$ | 기저 벡터를 열로 |
| 기저 변환 | $\mathbf{v}_{\mathcal{B}'} = M_{\mathcal{B}'}^{-1} M_{\mathcal{B}} \mathbf{v}_{\mathcal{B}}$ | 기저 간 좌표 변환 |

| 참조 문서 | 내용 |
|-----------|------|
| 《행렬 기초》 §7 | 직교 행렬 — 정규 직교 기저의 역행렬 |
| 《변환 행렬》 §5 | 좌표계 변환 — 월드↔로컬 |
| 《LookAt & 카메라》 | 카메라 좌표계 구성 |
| 《외적》 §3 | 법선 벡터 — TBN의 bitangent |
