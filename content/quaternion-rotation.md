---
title: 회전 & 보간 (Rotation & Interpolation)
slug: quaternion-rotation
---

## 소개

쿼터니언의 가장 큰 장점은 부드러운 회전 보간과 짐벌락(Gimbal Lock) 회피다. 이 페이지에서는 쿼터니언으로 회전을 표현하고, 오일러 각도와 상호 변환하며, Slerp로 보간하는 방법을 다룬다.

> **핵심**
> 오일러 각도는 직관적이지만 짐벌락이 있고, 쿼터니언은 짐벌락이 없고 보간이 부드럽다.

---

## 1. 오일러 각도 (Euler Angles)

세 개의 회전 각도(pitch, yaw, roll)로 3D 회전을 표현한다.

- **pitch** (Y축): 위/아래
- **yaw** (Z축): 좌/우
- **roll** (X축): 좌/우 기울임

### 오일러 각도 → 회전 행렬

$$\mathbf{R} = \mathbf{R}_z(\text{yaw}) \times \mathbf{R}_y(\text{pitch}) \times \mathbf{R}_x(\text{roll})$$

> **주의**: 회전 순서가 중요하다! (ZYX, XYZ 등 여러 관례가 있음)

### 장점

- 직관적이고 이해하기 쉬움
- 3개 값만 저장 (메모리 효율)
- UI에서 조작하기 편함

### 단점

- **짐벌락**: 특정 각도에서 회전 축이 중복되어 자유도 상실
- 회전 합성이 복잡
- 보간이 어려움 (각도를 직접 lerp하면 회전이 꼬임)

---

## 2. 짐벌락 (Gimbal Lock)

### 발생 원리

오일러 각도에서 pitch가 $\pm 90°$에 도달하면, yaw와 roll이 같은 회전을 만든다:

$$\mathbf{R}_z(\text{yaw}) \times \mathbf{R}_y(90°) \times \mathbf{R}_x(\text{roll})$$

$\mathbf{R}_y(90°)$가 X축(roll)과 Z축(yaw)을 정렬시켜 yaw와 roll이 같은 효과를 내어 자유도 1개가 상실된다.

### 시각적 예시

비행기가 정확히 위를 향할 때 (pitch = 90°):
- yaw를 돌리든 roll을 돌리든 같은 회전 발생
- → 한 축의 회전이 불가능해짐
- → 급격히 뒤집히거나 튕기는 현상

### 해결책: 쿼터니언

쿼터니언은 4차원 표현으로 3D 회전 공간 $SO(3)$을 특이점 없이 커버한다 → 짐벌락 없음.

> **주의**: 쿼터니언을 오일러 각도로 변환했다가 다시 쿼터니언으로 돌리면 짐벌락이 다시 나타난다. 변환 과정에서 정보가 손실될 수 있다.

---

## 3. 오일러 ↔ 쿼터니언 변환

### 오일러 → 쿼터니언

각 축별 쿼터니언을 생성하여 ZYX 순서로 합성한다:

$$\mathbf{q} = \mathbf{q}_y \times \mathbf{q}_p \times \mathbf{q}_r$$

```text
function fromEuler(pitch, yaw, roll):
    // 각 축별 쿼터니언 생성 (pitch=Y, yaw=Z, roll=X)
    qp = fromAxisAngle((0,1,0), pitch)
    qy = fromAxisAngle((0,0,1), yaw)
    qr = fromAxisAngle((1,0,0), roll)

    // ZYX 순서로 합성: yaw(Z) × pitch(Y) × roll(X)
    return qy × qp × qr
```

### 쿼터니언 → 오일러

**roll (x-axis rotation):**

$$\sin(\text{roll}) = 2 \cdot (w \cdot x + y \cdot z)$$

$$\cos(\text{roll}) = 1 - 2 \cdot (x^2 + y^2)$$

$$\text{roll} = \text{atan2}(\sin(\text{roll}), \cos(\text{roll}))$$

**pitch (y-axis rotation):**

$$\sin(\text{pitch}) = 2 \cdot (w \cdot y - z \cdot x)$$

$$\sin(\text{pitch}) = \text{clamp}(\sin(\text{pitch}), -1, 1)$$

$$\text{pitch} = \arcsin(\sin(\text{pitch}))$$

**yaw (z-axis rotation):**

$$\sin(\text{yaw}) = 2 \cdot (w \cdot z + x \cdot y)$$

$$\cos(\text{yaw}) = 1 - 2 \cdot (y^2 + z^2)$$

$$\text{yaw} = \text{atan2}(\sin(\text{yaw}), \cos(\text{yaw}))$$

```text
function toEuler(q):
    // roll (x-axis rotation)
    sinRoll = 2 × (w×x + y×z)
    cosRoll = 1 - 2 × (x×x + y×y)
    roll = atan2(sinRoll, cosRoll)

    // pitch (y-axis rotation)
    sinPitch = 2 × (w×y - z×x)
    sinPitch = clamp(sinPitch, -1, 1)
    pitch = asin(sinPitch)

    // yaw (z-axis rotation)
    sinYaw = 2 × (w×z + x×y)
    cosYaw = 1 - 2 × (y×y + z×z)
    yaw = atan2(sinYaw, cosYaw)

    return pitch, yaw, roll
```

> **주의**: 변환 시 짐벌락 영역(pitch ≈ ±90°)에서 yaw와 roll이 불안정해진다. 표시용으로만 사용하고, 내부 계산은 쿼터니언으로 유지하자.

---

## 4. Slerp (Spherical Linear Interpolation)

두 쿼터니언 사이를 구면 위에서 일정한 각속도로 보간한다.

### 공식

$$\text{dot} = w_0 \cdot w_1 + x_0 \cdot x_1 + y_0 \cdot y_1 + z_0 \cdot z_1$$

두 쿼터니언이 같은 회전을 나타내면 더 짧은 경로를 선택:

$$\text{if dot} < 0: \quad \mathbf{q}_1 = -\mathbf{q}_1, \quad \text{dot} = -\text{dot}$$

매우 가까우면 ($\text{dot} > 0.9995$) Nlerp로 대체 (수치 안정). 그렇지 않으면:

$$\theta_0 = \arccos(\text{dot})$$

$$\theta = \theta_0 \cdot t$$

$$s_0 = \cos(\theta) - \text{dot} \cdot \frac{\sin(\theta)}{\sin(\theta_0)}$$

$$s_1 = \frac{\sin(\theta)}{\sin(\theta_0)}$$

$$\text{slerp}(\mathbf{q}_0, \mathbf{q}_1, t) = s_0 \cdot \mathbf{q}_0 + s_1 \cdot \mathbf{q}_1$$

```text
slerp(q0, q1, t):
    dot = w0×w1 + x0×x1 + y0×y1 + z0×z1

    if dot < 0:
        q1 = -q1       // 더 짧은 경로 선택
        dot = -dot

    if dot > 0.9995:
        return normalize(lerp(q0, q1, t))

    theta0 = acos(dot)
    theta = theta0 × t
    sinTheta = sin(theta)
    sinTheta0 = sin(theta0)

    s0 = cos(theta) - dot × sinTheta / sinTheta0
    s1 = sinTheta / sinTheta0

    return s0 × q0 + s1 × q1
```

### 특징

- **등각속도**: 회전 속도가 일정
- **최단 경로**: 자동으로 더 짧은 회전 경로 선택
- 정확하지만 sin/cos 연산 필요

---

## 5. Nlerp (Normalized Lerp)

Lerp 후 정규화하는 간단한 근사 보간이다.

### 공식

$$\text{result} = (1 - t) \cdot \mathbf{q}_0 + t \cdot \mathbf{q}_1$$

$$\text{nlerp}(\mathbf{q}_0, \mathbf{q}_1, t) = \text{normalize}(\text{result})$$

```text
nlerp(q0, q1, t):
    result = lerp(q0, q1, t)     // (1-t)×q0 + t×q1
    return normalize(result)
```

### Slerp vs Nlerp

| 특징 | Slerp | Nlerp |
|------|-------|-------|
| 각속도 | 일정 | 비선형 (가운데서 빨라짐) |
| 속도 | 느림 | 빠름 |
| 정확도 | 정확 | 근사 |
| 사용 | 정밀 애니메이션 | 일반 회전 |

> **실전 팁**: 두 회전 사이의 각도가 작을 때(30도 이내)는 Nlerp와 Slerp가 거의 동일하다. 대부분의 게임에서는 Nlerp로 충분.

---

## 6. Double Cover (이중 덮개)

쿼터니언 $\mathbf{q}$와 $-\mathbf{q}$는 같은 회전을 나타낸다:

$$\mathbf{q} = (w, x, y, z) \quad \rightarrow \text{회전 } \theta$$

$$-\mathbf{q} = (-w, -x, -y, -z) \quad \rightarrow \text{같은 회전 } \theta$$

이유:

$$\mathbf{q} = (\cos(\theta/2), \sin(\theta/2) \cdot \mathbf{u})$$

$$-\mathbf{q} = (\cos(\theta/2 + \pi), \sin(\theta/2 + \pi) \cdot \mathbf{u})$$

둘 다 3D에서는 같은 회전이다.

### 실전 영향

보간 시 더 짧은 경로를 선택하기 위해 dot이 음수면 반전한다:

$$\text{dot} = \text{dot}(\mathbf{q}_0, \mathbf{q}_1)$$

$$\text{if dot} < 0: \quad \mathbf{q}_1 = -\mathbf{q}_1$$

> 이걸 안 하면 360도 돌아서 보간될 수 있음.

---

## 7. 회전 누적과 감쇠

### 각속도 기반 회전

```text
// 매 프레임 각속도를 쿼터니언에 누적
angularVelocity = (rx, ry, rz)    // 라디안/초
// fromEuler(pitch, yaw, roll) 시그니처에 맞춰 (Y, Z, X) 매핑
deltaRot = fromEuler(angularVelocity.y, angularVelocity.z, angularVelocity.x)
currentRot = deltaRot × currentRot    // 순서 주의!

currentRot = normalize(currentRot)   // 오차 보정
```

### 감쇠 회전 (Damping)

현재 회전을 목표 회전으로 부드럽게:

$$\text{currentRot} = \text{slerp}(\text{currentRot}, \text{targetRot}, 1 - \exp(-\text{damping} \cdot dt))$$

> **프레임 독립적 감쇠**: $1 - \exp(-\text{rate} \cdot dt)$ 공식을 사용하면 프레임 속도에 관계없이 일관된 감쇠 속도를 얻는다.

---

## 8. 실전 예시: 적이 플레이어를 향해 회전

```text
// 현재 방향과 목표 방향
currentDir = enemyForward
targetDir = normalize(player.position - enemy.position)

// 두 방향 사이의 회전 쿼터니언
fromQ = fromDirection(currentDir)
toQ = fromDirection(targetDir)

// 부드럽게 회전 (프레임 독립적)
t = 1 - exp(-turnSpeed × dt)
enemy.rotation = slerp(fromQ, toQ, t)

// forward 벡터 업데이트
enemyForward = rotate(enemy.rotation, (0, 0, 1))
```

### fromDirection 함수

> 참고: 아래는 Z축을 forward로 가정하는 방향 헬퍼로, 위 오일러 축 라벨과는 독립적이다.

```text
function fromDirection(dir):
    // (0, 0, 1)을 dir 방향으로 회전하는 쿼터니언
    defaultForward = (0, 0, 1)
    dot = dot(defaultForward, dir)

    if dot > 0.9999:
        return (1, 0, 0, 0)       // 같은 방향, 항등 회전
    if dot < -0.9999:
        // 정반대 → 임의의 축으로 180도
        return fromAxisAngle((1, 0, 0), 180°)

    axis = cross(defaultForward, dir)
    angle = acos(dot)
    return fromAxisAngle(axis, angle)
```

---

## 9. 실전 예시: 카메라 짐벌락 방지 (1인칭)

```text
// 오일러 각도 기반 (짐벌락 위험)
yaw += mouseDeltaX × sensitivity
pitch += mouseY × sensitivity
if pitch > 89°: pitch = 89°      // 클램핑으로 방지
if pitch < -89°: pitch = -89°

// 쿼터니언으로 변환 (정확한 회전)
q_pitch = fromAxisAngle((0, 1, 0), pitch)   // pitch = Y축
q_yaw = fromAxisAngle((0, 0, 1), yaw)        // yaw = Z축
cameraRot = q_yaw × q_pitch       // yaw(Z) 후 pitch(Y): ZYX 순서

// 클램핑만으로도 대부분의 상황에서 충분하지만,
// 복잡한 카메라 시스템에서는 쿼터니언 연산만으로 처리하는 것이 안전
```

> **관례**: 위 예시는 Z-up 월드와 본 문서의 ZYX 관례를 따른다(pitch=Y축, yaw=Z축). Y-up 월드(《LookAt》 §7)라면 yaw→Y축, pitch→X축으로 바꿔야 한다.

---

## 10. 쿼터니언 → 회전 행렬 변환

내부 계산은 쿼터니언으로 하되, 셰이더나 버퍼에 넘길 때는 회전 행렬로 변환해야 한다 (《쿼터니언 기초》 §8의 권장 사항). 단위 쿼터니언 $\mathbf{q}=(w,x,y,z)$에 대해, column-vector $\mathbf{v}' = R\mathbf{v}$ 기준으로 회전 행렬은:

$$
R = \begin{bmatrix}
1 - 2(y^2 + z^2) & 2(xy - wz) & 2(xz + wy) \\
2(xy + wz) & 1 - 2(x^2 + z^2) & 2(yz - wx) \\
2(xz - wy) & 2(yz + wx) & 1 - 2(x^2 + y^2)
\end{bmatrix}
$$

### 동차 4×4 행렬

위 3×3 회전 행렬을 좌상단에 배치하고, 마지막 행·열을 $(0, 0, 0, 1)$로 채운다:

$$
M = \begin{bmatrix}
R_{00} & R_{01} & R_{02} & 0 \\
R_{10} & R_{11} & R_{12} & 0 \\
R_{20} & R_{21} & R_{22} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

> **활용**: 이동·스케일이 없는 순수 회전의 최종 변환 행렬이다. 이동·스케일을 함께 합치려면 $T \times R \times S$로 조립한다 (《변환 행렬》 §4).

**게임에서의 활용:**
```text
function quaternionToMatrix(q):
    // q = (w, x, y, z), 단위 쿼터니언 가정
    return [
        1 - 2×(y×y + z×z),  2×(x×y - w×z),      2×(x×z + w×y),      0,
        2×(x×y + w×z),      1 - 2×(x×x + z×z),  2×(y×z - w×x),      0,
        2×(x×z - w×y),      2×(y×z + w×x),      1 - 2×(x×x + y×y),  0,
        0,                  0,                  0,                  1
    ]
```

---

## 11. 빠른 참조

| 연산 | 공식/설명 |
|------|----------|
| 오일러→쿼터니언 | fromAxisAngle × 합성 |
| 쿼터니언→오일러 | $\text{atan2} + \arcsin$ 조합 |
| 쿼터니언→행렬 | 위 $R$ (셰이더/엔진 전달, §10) |
| Slerp | $\sin$ 기반 구면 보간 |
| Nlerp | $\text{normalize}(\text{lerp}(\mathbf{q}_0, \mathbf{q}_1, t))$ |
| 더 짧은 경로 | $\text{dot} < 0 \rightarrow \mathbf{q}_1 = -\mathbf{q}_1$ |
| 각속도 누적 | $\Delta\mathbf{q} \times \mathbf{q}_{\text{current}}$ |
| 감쇠 | $\text{slerp}(\mathbf{q}_{\text{cur}}, \mathbf{q}_{\text{target}}, 1 - \exp(-\text{rate} \cdot dt))$ |
| 짐벌락 방지 | 쿼터니언 사용, pitch 클램핑 |