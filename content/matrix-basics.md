---
title: 행렬 기초 (Matrix Basics)
slug: matrix-basics
---

## 소개

행렬은 게임 수학에서 좌표계 변환, 객체 이동/회전/스케일, 카메라 투영 등을 표현하는 핵심 도구다. 3D 그래픽스의 모든 변환은 행렬 곱셈으로 이루어진다.

> **핵심**
> 행렬은 다차원 선형 변환을 표현하는 격자이고, 행렬 곱셈은 변환의 합성이다.

---

## 1. 행렬의 정의

$m \times n$ 행렬은 $m$개의 행(row)과 $n$개의 열(column)로 구성된 숫자 배열이다.

$$
M = \begin{bmatrix}
m_{00} & m_{01} & m_{02} & m_{03} \\
m_{10} & m_{11} & m_{12} & m_{13} \\
m_{20} & m_{21} & m_{22} & m_{23} \\
m_{30} & m_{31} & m_{32} & m_{33}
\end{bmatrix}
$$

게임에서 주로 사용하는 행렬:
- **2×2**: 2D 회전
- **3×3**: 3D 회전, 스케일
- **4×4**: 3D 변환 (이동 + 회전 + 스케일) — **가장 많이 사용**
- **3×1 / 4×1**: 벡터 (열 벡터 표현)

> **행 벡터 vs 열 벡터**
> 게임 엔진마다 관례가 다르다:
> - **열 벡터 (OpenGL, Vulkan)**: $v' = M \times v$
> - **행 벡터 (DirectX 레거시 관례)**: $v' = v \times M$ (Unity 공식 문서 관례는 열 벡터 $v' = M \times v$)
> 이 문서에서는 열 벡터 방식을 기본으로 사용한다.

---

## 2. 단위 행렬 (Identity Matrix)

대각선이 1이고 나머지가 0인 행렬. 어떤 벡터에 곱해도 변하지 않는다.

$$
I = \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

$$I \times v = v$$
$$I \times M = M \times I = M$$

---

## 3. 행렬 곱셈 (Matrix Multiplication)

두 행렬을 곱하면 변환이 합성된다.

### 공식 (4×4 × 4×4)

$$C = A \times B$$

$$C_{ij} = \sum_{k=0}^{3} A_{ik} \times B_{kj}$$

### 예시 (2×2)

$$
A \times B = \begin{bmatrix} a_{00} & a_{01} \\ a_{10} & a_{11} \end{bmatrix} \times \begin{bmatrix} b_{00} & b_{01} \\ b_{10} & b_{11} \end{bmatrix}
$$

$$
= \begin{bmatrix}
a_{00} \times b_{00} + a_{01} \times b_{10} & a_{00} \times b_{01} + a_{01} \times b_{11} \\
a_{10} \times b_{00} + a_{11} \times b_{10} & a_{10} \times b_{01} + a_{11} \times b_{11}
\end{bmatrix}
$$

### 중요한 성질

| 성질 | 수식 | 설명 |
|------|------|------|
| 결합법칙 | $(A \times B) \times C = A \times (B \times C)$ | 괄호(결합) 순서만 자유, 피연산자 나열 순서는 고정 ($AB \neq BA$) |
| **비교환법칙** | $A \times B \neq B \times A$ | **순서 중요!** |
| 분배법칙 | $A \times (B+C) = A \times B + A \times C$ | 덧셈에 대해 분배 |

> **주의**: 행렬 곱셈은 교환법칙이 성립하지 않는다. "먼저 회전하고 이동" vs "먼저 이동하고 회전"은 완전히 다른 결과를 낳는다.

**게임에서의 활용:**
```text
// 부모 → 자식 계층 변환
worldMatrix = parentMatrix × localMatrix

// 모델 → 뷰 → 투영
finalPosition = projection × view × model × vertexPosition
```

---

## 4. 전치 행렬 (Transpose)

행과 열을 바꾼 행렬이다.

$$
M^T = \begin{bmatrix} a & b \\ c & d \end{bmatrix}^T = \begin{bmatrix} a & c \\ b & d \end{bmatrix}
$$

$$
M^T = \begin{bmatrix} a & b & c \\ d & e & f \end{bmatrix}^T = \begin{bmatrix} a & d \\ b & e \\ c & f \end{bmatrix}
$$

### 성질

$$(A \times B)^T = B^T \times A^T$$
$$(A^T)^T = A$$
$$(A + B)^T = A^T + B^T$$

**게임에서의 활용:**
- **법선 행렬**: 조명 계산 시 $(M^{-1})^T$ 사용
- **직교 행렬**: 회전 행렬의 역행렬 = 전치 행렬 (빠른 역행렬)

---

## 5. 역행렬 (Inverse Matrix)

역행렬 $M^{-1}$은 $M$의 변환을 되돌리는 행렬이다.

$$M \times M^{-1} = M^{-1} \times M = I$$

### 2×2 역행렬

$$
A^{-1} = \frac{1}{ad - bc} \begin{bmatrix} d & -b \\ -c & a \end{bmatrix}
$$

> $(ad - bc)$는 행렬식(determinant)이다

### 4×4 일반 역행렬

일반적인 4×4 역행렬은 계산이 복잡하므로 게임 엔진에서는 보통 라이브러리 함수를 사용한다. 하지만 특수한 경우는 빠른 공식이 있다:

$$R^{-1} = R^T \quad \text{(순수 회전 행렬의 역행렬 = 전치 행렬)}$$

$$T(\mathbf{t})^{-1} = T(-\mathbf{t}) \quad \text{(순수 이동 행렬의 역행렬 = 이동 부호 반전)}$$

### 강체 변환 (이동 + 회전)의 고속 역행렬

스케일이 없고 이동과 회전만 있는 행렬($M = T \times R$, 예: 카메라 월드 행렬, 강체 오브젝트)은 일반 4×4 역행렬을 계산할 필요 없이 다음과 같이 즉시 구해진다:

$$
\begin{bmatrix}
\mathbf{R} & \mathbf{t} \\
\mathbf{0}^T & 1
\end{bmatrix}^{-1}
=
\begin{bmatrix}
\mathbf{R}^T & -\mathbf{R}^T \mathbf{t} \\
\mathbf{0}^T & 1
\end{bmatrix}
$$

> 카메라 월드 변환을 뒤집어 뷰 행렬(View Matrix)을 만들 때 바로 이 공식을 사용한다 (LookAt의 이동부가 $-\mathbf{R}^T \cdot \mathbf{eye}$가 되는 이유다).

$$M = T \times R \times S \quad \Longrightarrow \quad M^{-1} = S^{-1} \times R^T \times T^{-1}$$

**게임에서의 활용:**
- **월드 → 로컬 변환**: 부모 공간으로 변환 되돌리기
- **광선 교차**: 월드 공간 광선을 객체 공간으로 변환
- **빌보드**: 카메라 행렬의 역으로 객체를 카메라에 정면으로

---

## 6. 행렬식 (Determinant)

정방 행렬의 행렬식 $\det(M)$ 또는 $|M|$은 변환이 공간을 얼마나 확대/축소하는지를 나타낸다.

### 2×2

$$|A| = a \times d - b \times c$$

### 3×3 (Sarrus 법)

$$|A| = a(ei - fh) - b(di - fg) + c(dh - eg)$$

### 4×4 (여인자 전개)

4×4는 라플라스 전개로 계산하며, 게임에서는 라이브러리를 사용한다.

### 행렬식의 의미

| 값 | 의미 |
|----|------|
| $> 0$ | 방향 유지 (오른손 좌표계 유지) |
| $< 0$ | 방향 반전 (반사/미러링) |
| $= 0$ | 변환이 축소 → **역행렬 없음** (특이 행렬) |

**게임에서의 활용:**
- **역행렬 존재 여부**: $\det \neq 0$이어야 역행렬 존재
- **볼륨 변화**: $\det = 2$면 공간이 2배 확대
- **반사 검출**: $\det < 0$이면 미러링 (법선 반전 필요)

---

## 7. 직교 행렬 (Orthogonal Matrix)

회전 행렬은 직교 행렬이다. 직교 행렬의 성질:

$$M^T \times M = I \quad \text{(전치 = 역행렬)}$$

- 행 벡터들이 서로 직교하고 단위 벡터
- 열 벡터들도 서로 직교하고 단위 벡터

$$\det(M) = \pm 1 \quad \text{(+1이면 순수 회전, -1이면 반사)}$$

> **실전 팁**: 회전만 포함된 행렬은 전치만으로 역행렬을 구할 수 있다. 일반적인 4×4 역행렬 계산(약 100개 곱셈) 대신 전치(0개 곱셈)로 처리할 수 있어 엄청나게 빠르다.

---

## 8. 동차 좌표 (Homogeneous Coordinates)

3D 점을 4D 벡터 $(x, y, z, w)$로 표현하는 체계이다.

$$\text{점 (Position)}: (x, y, z, 1) \quad \text{($w = 1$ → 이동 변환에 영향받음)}$$

$$\text{방향 (Vector)}: (x, y, z, 0) \quad \text{($w = 0$ → 이동 변환 무시)}$$

### 원근 분할 (Perspective Divide)

$$\text{result} = \left(\frac{x}{w}, \frac{y}{w}, \frac{z}{w}\right) \quad \text{(4D → 3D 변환, $w$로 나눔)}$$

**게임에서의 활용:**
- **투영 변환**: 원근 투영 후 $w$로 나누어 원근 효과
- **이동과 방향 구분**: 점은 이동되고, 방향 벡터는 이동 무시
- **선형 보간**: $w$ 값으로 투영 공간에서 올바른 보간

---

## 9. 실전 예시: 모델 → 월드 → 뷰 → 클립 변환

3D 파이프라인의 핵심 변환 체인:

```text
// 1. 모델 변환 (객체 → 월드)
modelMatrix = translate(position) × rotate(rotation) × scale(size)

// 2. 뷰 변환 (월드 → 카메라)
viewMatrix = inverse(cameraTransform)

// 3. 투영 변환 (카메라 → 클립 공간)
clipPosition = projectionMatrix × viewMatrix × modelMatrix × vertexPosition

// 4. 원근 분할 (클립 → NDC)
ndcPosition = clipPosition.xyz / clipPosition.w
```

---

## 10. 빠른 참조

| 연산 | 수식 | 활용 |
|------|------|------|
| 단위 행렬 | $I$ (대각선 1) | 기준 |
| 곱셈 | $C_{ij} = \sum A_{ik} \cdot B_{kj}$ | 변환 합성 |
| 전치 | $M^T$ (행↔열 교환) | 역회전, 법선 행렬 |
| 역행렬 | $M \times M^{-1} = I$ | 변환 되돌리기 |
| 행렬식 | $\|M\|$ | 볼륨 변화, 역행렬 존재 |
| 동차 좌표 | $(x,y,z,w)$ | 점($w=1$)/방향($w=0$) |
| 원근 분할 | $(x/w, y/w, z/w)$ | NDC 변환 |