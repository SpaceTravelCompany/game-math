---
title: LookAt & 카메라 (LookAt & Camera)
slug: look-at-camera
---

## 소개

LookAt은 카메라(또는 객체)가 특정 위치를 바라보도록 하는 변환 행렬을 만드는 기법이다. 3D 게임에서 카메라 시스템, 적 AI의 시선 처리, 빌보드(billboard) 효과 등에 필수적으로 사용된다.

> **핵심**
> LookAt 행렬 = 카메라 위치, 타겟 위치, Up 벡터로부터 forward/right/up 축을 계산해 만드는 뷰 행렬이다.

---

## 1. LookAt 행렬 구성

세 가지 입력으로 카메라의 좌표계를 구성한다:

- **eye**: 카메라 위치
- **target**: 바라볼 위치
- **up**: 위쪽 방향 (보통 (0,1,0))

### 축 계산

$$
\mathbf{forward} = \text{normalize}(\mathbf{eye} - \mathbf{target}) \quad \text{(주의: -Z 방향)}
$$

> **주의**: 여기서 forward는 카메라가 바라보는 방향(-Z, 타깃에서 멀어지는 방향). **《외적》 문서 §7**의 forward는 타깃을 향하는 방향(+Z)으로 정의가 반대이므로 문맥을 구분할 것.

$$
\mathbf{right} = \text{normalize}(\mathbf{forward} \times \mathbf{up})
$$

$$
\mathbf{cameraUp} = \mathbf{right} \times \mathbf{forward}
$$

### 뷰 행렬 (View Matrix)

$$
\mathbf{LookAt} = \begin{bmatrix}
\mathbf{right}_x & \mathbf{right}_y & \mathbf{right}_z & -\mathbf{right} \cdot \mathbf{eye} \\
\mathbf{cameraUp}_x & \mathbf{cameraUp}_y & \mathbf{cameraUp}_z & -\mathbf{cameraUp} \cdot \mathbf{eye} \\
\mathbf{forward}_x & \mathbf{forward}_y & \mathbf{forward}_z & -\mathbf{forward} \cdot \mathbf{eye} \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

> **오른손 좌표계 기준**. 왼손 좌표계(DirectX/Unity)에서는 forward 방향이나 축 계산 순서가 다를 수 있다.

---

## 2. 카메라 공간 (Camera/View Space)

뷰 행렬은 월드 공간을 카메라 공간으로 변환한다.

### 카메라 공간의 특징

```text
- 카메라가 원점 (0, 0, 0)
- 카메라가 -Z 방향을 바라봄 (오른손 좌표계)
- +X가 오른쪽, +Y가 위쪽
```

$$
\mathbf{viewPos} = \mathbf{viewMatrix} \times \mathbf{worldPos}
$$

### 왜 카메라는 -Z를 바라보는가?

역사적 이유로 OpenGL은 오른손 좌표계에서 카메라가 -Z를 바라본다. 이 관례가 대부분의 그래픽스 API에 전파되었다. DirectX는 +Z를 바라보지만, 최신 엔진들은 추상화되어 개발자가 직접 다룰 일이 적다.

```text
// 카메라 공간 (오른손 좌표계)
     +Y (up)
      |
      |
      o—— +X (right)
     /
    / -Z (forward, 카메라가 바라보는 방향)
```

---

## 3. Up 벡터의 역할과 문제

### Up 벡터가 필요한 이유

forward 벡터만으로는 카메라의 롤(roll)을 결정할 수 없다. up 벡터는 "어느 방향이 위인지" 알려준다.

### Gimbal Lock 상황

$$
\mathbf{forward} = \text{normalize}(\mathbf{eye} - \mathbf{target})
$$

$$
\mathbf{right} = \mathbf{forward} \times \mathbf{up} \quad (\mathbf{forward} \parallel \mathbf{up} \Rightarrow \mathbf{right} = \mathbf{0})
$$

이 경우 카메라 행렬이 붕괴됨

### 해결책

```text
// 1. 카메라가 정확히 위/아래를 볼 때 다른 up 사용
if abs(dot(forward, worldUp)) > 0.999:
    up = (0, 0, 1)    // 대체 up 벡터
else:
    up = worldUp

// 2. 카메라 자체의 up 벡터 유지 (프리 카메라)
cameraUp = currentCameraUp   // 이전 프레임의 up 유지
```

---

## 4. View 행렬 = 역행렬

뷰 행렬은 카메라의 월드 변환 행렬의 역행렬이다.

$$
\mathbf{cameraWorld} = \mathbf{T}(\mathbf{eye}) \times \mathbf{R}(\text{lookAtOrientation})
$$

$$
\mathbf{viewMatrix} = \mathbf{cameraWorld}^{-1}
$$

### LookAt으로 직접 계산 (빠른 방법)

역행렬을 일반적으로 계산하는 것보다, LookAt 공식으로 직접 만드는 것이 빠르다:

```text
// 회전 부분은 전치만으로 역행렬 (직교 행렬)
// 이동 부분은 -R^T × eye
```

$$
\mathbf{viewMatrix} = \begin{bmatrix}
\mathbf{R}^T & -\mathbf{R}^T \cdot \mathbf{eye} \\
\mathbf{0} & 1
\end{bmatrix}
$$

---

## 5. 3인칭 카메라

플레이어 뒤에서 따라오는 카메라 시스템.

### 기본 구현

```text
// 카메라 위치 = 플레이어 위치 + 뒤쪽 오프셋
offset = (0, height, -distance)
cameraPos = playerPos + rotateY(playerYaw) × offset
target = playerPos + (0, lookHeight, 0)

viewMatrix = lookAt(cameraPos, target, worldUp)
```

### 충돌 처리

```text
// 카메라가 벽에 안 끼이게 레이캐스트
rayDir = normalize(cameraPos - playerPos)
rayResult = raycast(playerPos, rayDir, desiredDistance)

if rayResult.hit:
    cameraPos = rayResult.point + normal × 0.2  // 벽에서 살짝 떼기
else:
    cameraPos = playerPos + offset

viewMatrix = lookAt(cameraPos, playerPos, worldUp)
```

---

## 6. 빌보드 (Billboard)

객체가 항상 카메라를 바라보게 만드는 기법이다.

### 파티클 빌보드

```text
// 카메라의 right와 up 벡터 추출
right = viewMatrix.row(0)    // 뷰 행렬의 첫 번째 행
up = viewMatrix.row(1)       // 두 번째 행

// 파티클 위치에서 카메라를 향하는 쿼드
center = particle.position
halfSize = particle.size / 2
```

$$
\mathbf{v_0} = \mathbf{center} + \mathbf{right} \times (-\text{halfSize}) + \mathbf{up} \times (-\text{halfSize})
$$

$$
\mathbf{v_1} = \mathbf{center} + \mathbf{right} \times (\text{halfSize}) + \mathbf{up} \times (-\text{halfSize})
$$

$$
\mathbf{v_2} = \mathbf{center} + \mathbf{right} \times (\text{halfSize}) + \mathbf{up} \times (\text{halfSize})
$$

$$
\mathbf{v_3} = \mathbf{center} + \mathbf{right} \times (-\text{halfSize}) + \mathbf{up} \times (\text{halfSize})
$$

### Y축 빌보드 (나무, 캐릭터)

```text
// 카메라가 돌아도 Y축으로만 회전 (나무 등)
toCamera = cameraPos - billboardPos
toCamera.y = 0    // Y축 무시
```

$$
\text{yaw} = \arctan(\mathbf{toCamera}_x, \mathbf{toCamera}_z)
$$

$$
\mathbf{billboardMatrix} = \mathbf{T}(\mathbf{billboardPos}) \times \mathbf{R}_y(\text{yaw}) \times \mathbf{S}(\text{scale})
$$

---

## 7. 카메라 기법 총정리

### 고정 카메라 (Fixed)

```text
viewMatrix = lookAt(fixedPos, fixedTarget, worldUp)
```

### 1인칭 (FPS)

```text
// 마우스로 pitch/yaw 제어
yaw += mouseDeltaX × sensitivity
pitch += mouseDeltaY × sensitivity
pitch = clamp(pitch, -89°, 89°)    // 짐벌락 방지
```

$$
\mathbf{forward} = \begin{pmatrix}
\cos(\text{pitch}) \cdot \sin(\text{yaw}) \\
\sin(\text{pitch}) \\
-\cos(\text{pitch}) \cdot \cos(\text{yaw})
\end{pmatrix}
$$

```text
cameraPos = playerPos + eyeHeight
viewMatrix = lookAt(cameraPos, cameraPos + forward, worldUp)
```

### 3인칭 (Orbit)

```text
// 마우스 드래그로 궤도 회전
yaw += mouseDeltaX × sensitivity
pitch += mouseDeltaY × sensitivity
pitch = clamp(pitch, minPitch, maxPitch)

offset = spherical(distance, pitch, yaw)
cameraPos = target + offset
viewMatrix = lookAt(cameraPos, target, worldUp)
```

### RTS/전략 카메라

```text
// 위에서 내려다보는 고정 각도
pitch = 60°    // 고정
cameraPos = target + spherical(distance, pitch, yaw)
viewMatrix = lookAt(cameraPos, target, worldUp)
```

---

## 8. 실전 예시: 카메라 흔들림 (Camera Shake)

```text
// 트라우마(trauma) 기반 카메라 흔들림
trauma = 1.0    // 0~1, 충돌 시 증가
trauma = max(0, trauma - dt × recoveryRate)
```

$$
\text{shake} = \text{trauma}^2 \quad \text{(비선형: trauma가 낮으면 거의 안 흔들림)}
$$

$$
\text{offsetX} = (\text{noise}(\text{time} \times \text{freq}) \times 2 - 1) \times \text{maxShake} \times \text{shake}
$$

$$
\text{offsetY} = (\text{noise}(\text{time} \times \text{freq} + 100) \times 2 - 1) \times \text{maxShake} \times \text{shake}
$$

```text
finalPos = baseCameraPos + (offsetX, offsetY, 0)
viewMatrix = lookAt(finalPos, target, worldUp)
```

---

## 9. 빠른 참조

| 요소 | 공식/설명 |
|------|----------|
| Forward | normalize(eye - target) |
| Right | normalize(cross(forward, up)) |
| Up | cross(right, forward) |
| View 행렬 | [R^T, -R^T × eye; 0, 1] |
| 빌보드 | 뷰 행렬의 right/up 사용 |
| 짐벌락 방지 | forward ∥ up 체크 |
| 1인칭 | pitch clamping (±89°) |
| 3인칭 | spherical coordinates |
| 카메라 흔들림 | trauma² × noise |