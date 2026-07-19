---
title: 변환 행렬 (Transform Matrices)
slug: transforms
---

## 소개

변환 행렬은 객체의 이동(Translation), 회전(Rotation), 스케일(Scale)을 하나의 4×4 행렬로 표현한다. 게임 엔진의 모든 객체 위치, 카메라, 조명, 애니메이션 본(Bone)이 이 행렬을 기반으로 동작한다.

> **핵심**
> TRS 행렬 = $T$(이동) $\times$ $R$(회전) $\times$ $S$(스케일). 곱하는 순서가 결과를 바꾼다.

---

## 1. 이동 행렬 (Translation)

객체를 $(t_x, t_y, t_z)$만큼 이동시킨다.

$$
T(\mathbf{t}) = \begin{bmatrix}
1 & 0 & 0 & t_x \\
0 & 1 & 0 & t_y \\
0 & 0 & 1 & t_z \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

### 적용

$$
T \times v = \begin{bmatrix}
x + t_x \\
y + t_y \\
z + t_z \\
1
\end{bmatrix}
$$

> 점 $(x, y, z, 1)$을 $(t_x, t_y, t_z)$만큼 이동

> **주의**: 방향 벡터 ($w=0$)는 이동 영향을 받지 않는다. 이것이 동차 좌표의 핵심이다.

### 역행렬

$$T(\mathbf{t})^{-1} = T(-\mathbf{t})$$

---

## 2. 스케일 행렬 (Scale)

객체를 $(s_x, s_y, s_z)$배로 확대/축소한다.

$$
S(\mathbf{s}) = \begin{bmatrix}
s_x & 0 & 0 & 0 \\
0 & s_y & 0 & 0 \\
0 & 0 & s_z & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

### 비균일 스케일의 문제

```text
// 비균일 스케일 (sx ≠ sy ≠ sz)은 법선을 왜곡한다
// 법선에는 (M⁻¹)^T를 사용해야 함
normalMatrix = transpose(inverse(modelMatrix))
correctNormal = normalMatrix × normal
```

**게임에서의 활용:**
- **객체 크기**: 캐릭터, 환경 에셋 크기 조정
- **반사/거울**: 스케일 -1로 미러링
- **애니메이션**: 스쿼시 앤 스트레치 효과

### 역행렬

$$S(\mathbf{s})^{-1} = S\left(\frac{1}{s_x}, \frac{1}{s_y}, \frac{1}{s_z}\right)$$

---

## 3. 회전 행렬 (Rotation)

### 2D 회전

z축 기준으로 $\theta$만큼 회전:

$$
R(\theta) = \begin{bmatrix}
\cos\theta & -\sin\theta \\
\sin\theta & \cos\theta
\end{bmatrix}
$$

### 3D 회전 — 축별 회전

**X축 회전:**

$$
R_x(\theta) = \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & \cos\theta & -\sin\theta & 0 \\
0 & \sin\theta & \cos\theta & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

**Y축 회전:**

$$
R_y(\theta) = \begin{bmatrix}
\cos\theta & 0 & \sin\theta & 0 \\
0 & 1 & 0 & 0 \\
-\sin\theta & 0 & \cos\theta & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

**Z축 회전:**

$$
R_z(\theta) = \begin{bmatrix}
\cos\theta & -\sin\theta & 0 & 0 \\
\sin\theta & \cos\theta & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

> **기하 직관**: 회전 행렬의 각 열 벡터는 변환된 기저 벡터, 즉 원래 x/y/z 축이 회전 후 어디를 향하는지를 나타낸다. 예를 들어 $R$의 첫 번째 열은 원래 x축 단위벡터 $(1,0,0)$이 회전 후 도달하는 방향이다.

### 임의 축 회전 (Rodrigues' Rotation)

단위 벡터 $\mathbf{u}$를 축으로 $\theta$만큼 회전:

$$R(\mathbf{u}, \theta) = \cos\theta \cdot I + (1 - \cos\theta)(\mathbf{u} \otimes \mathbf{u}) + \sin\theta \cdot [\mathbf{u}]_\times$$

여기서 $[\mathbf{u}]_\times$는 $\mathbf{u}$의 반대칭 행렬 (외적 행렬):

$$
[\mathbf{u}]_\times = \begin{bmatrix}
0 & -u_z & u_y \\
u_z & 0 & -u_x \\
-u_y & u_x & 0
\end{bmatrix}
$$

### 역행렬

$$R(\theta)^{-1} = R(-\theta) = R(\theta)^T \quad \text{(전치 = 역행렬, 직교 행렬)}$$

---

## 4. TRS 합성 행렬

이동, 회전, 스케일을 하나의 행렬로 합성한다.

### 공식

$$M = T \times R \times S$$

### 결과 (4×4)

$$
M = \begin{bmatrix}
s_x \cdot r_{00} & s_y \cdot r_{01} & s_z \cdot r_{02} & t_x \\
s_x \cdot r_{10} & s_y \cdot r_{11} & s_z \cdot r_{12} & t_y \\
s_x \cdot r_{20} & s_y \cdot r_{21} & s_z \cdot r_{22} & t_z \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

여기서 $r_{00} \sim r_{22}$는 회전 행렬의 3×3 부분이다.

### 순서의 중요성

```text
// 올바른 순서: 먼저 스케일 → 회전 → 이동
M = T × R × S

// 잘못된 순서 예:
M = S × R × T    // 이동이 스케일에 영향받아 객체가 날아감
M = R × T × S    // 이동이 회전에 영향받아 위치가 틀어짐
```

> **원리**: 모델 공간에서 월드 공간으로 갈 때, 가장 안쪽(오른쪽) 변환부터 적용된다. 즉 $S \to R \to T$ 순서로 객체에 적용된다.

---

## 5. 좌표계 변환

객체 공간 → 월드 공간 → 카메라 공간으로 변환한다.

### 모델 행렬 (Model Matrix)

```text
// 로컬 → 월드
worldPos = modelMatrix × localPos

// 부모-자식 계층
worldMatrix = parentWorldMatrix × localMatrix
```

### 뷰 행렬 (View Matrix)

```text
// 월드 → 카메라 공간
viewMatrix = inverse(cameraWorldMatrix)
viewPos = viewMatrix × worldPos    // 결과: 월드 좌표 → 카메라 공간 좌표
```

### 전체 체인

$$\text{clipPos} = P \times V \times M \times \text{localPos}$$

```text
// 로컬 → 클립 공간
clipPos = projection × view × model × localPos
```

---

## 6. 행렬 분해 (Decomposition)

TRS 행렬에서 이동, 회전, 스케일을 다시 추출한다.

```text
// 이동 추출
translation = (M[0][3], M[1][3], M[2][3])

// 3×3 부분 행렬에서 스케일 추출
sx = length(M의 첫 번째 열 벡터)
sy = length(M의 두 번째 열 벡터)
sz = length(M의 세 번째 열 벡터)

// 회전 추출 (정규화)
R[i][j] = M[i][j] / scale[j]
```

> **주의**: 전단(shear) 변환이 포함된 경우 회전과 스케일 분해가 모호해진다 (전단과 회전을 구분할 수 없음). 비균일 스케일만 있는 경우는 각 열 벡터의 길이로 스케일을 추출하고 정규화하면 회전을 정확히 분해할 수 있다.

---

## 7. 변환 합성의 실전 예시

### 계층적 애니메이션 (Skeletal Animation)

```text
// 부모 본(예: 상완) → 자식 본(예: 하완)
parentMatrix = T(shoulderPos) × R(shoulderRotation)
childLocalMatrix = T(elbowOffset) × R(elbowRotation)
childWorldMatrix = parentMatrix × childLocalMatrix

// 최종 정점 위치
finalPos = childWorldMatrix × restPoseVertex
```

### 카메라 따라가기

```text
// 카메라를 플레이어 뒤에 배치
offset = T(0, 5, -10)  // 플레이어 뒤쪽, 위쪽
cameraMatrix = playerMatrix × offset
viewMatrix = inverse(cameraMatrix)
```

---

## 8. 2D 변환

2D 게임에서는 3×3 행렬로 동일한 변환을 표현한다.

$$
T = \begin{bmatrix}
1 & 0 & t_x \\
0 & 1 & t_y \\
0 & 0 & 1
\end{bmatrix}
$$

$$
R = \begin{bmatrix}
\cos\theta & -\sin\theta & 0 \\
\sin\theta & \cos\theta & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

### 2D TRS

$$M = T \times R \times S$$

$$
M = \begin{bmatrix}
s_x \cdot \cos\theta & -s_y \cdot \sin\theta & t_x \\
s_x \cdot \sin\theta & s_y \cdot \cos\theta & t_y \\
0 & 0 & 1
\end{bmatrix}
$$

---

## 9. 빠른 참조

| 변환 | 행렬 | 역행렬 |
|------|------|--------|
| 이동 $T(\mathbf{t})$ | 대각선 1 + 마지막 열 $\mathbf{t}$ | $T(-\mathbf{t})$ |
| 스케일 $S(\mathbf{s})$ | 대각선 $\mathbf{s}$ | $S(1/\mathbf{s})$ |
| 회전 $R(\theta)$ | cos/sin 배치 | $R(-\theta) = R^T$ |
| TRS | $T \times R \times S$ | $S^{-1} \times R^T \times T^{-1}$ |
| 계층 | parent $\times$ local | inverse(parent) $\times$ world |
| 뷰 | inverse(camera) | camera world |