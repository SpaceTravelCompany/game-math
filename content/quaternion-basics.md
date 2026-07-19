---
title: 쿼터니언 기초 (Quaternion Basics)
slug: quaternion-basics
---

## 소개

쿼터니언은 3D 회전을 표현하는 가장 효율적이고 강력한 수학 도구다. 오일러 각도의 짐벌락 문제를 해결하고, 회전 보간(Slerp)이 자연스러우며, 행렬보다 저장 공간이 적게 든다.

> **핵심**
> 쿼터니언은 4개의 숫자로 3D 회전을 표현하며, 짐벌락이 없고 보간이 부드럽다.

---

## 1. 정의

```text
       ↑ u (회전축, 단위벡터)
       │        ● v' (회전 후)
       │      ╱
       │     ╱ θ
       │    ╱
       ●───→ v (회전 전)

  3D 회전은 축(방향 3숫자) + 각도(1숫자)로 표현된다.
```

쿼터니언은 하나의 실수부와 세 개의 허수부로 구성된 복소수의 확장이다.

$$\mathbf{q} = w + xi + yj + zk = (w, x, y, z) = (w, \mathbf{v})$$

여기서 $\mathbf{v} = (x, y, z)$는 벡터부이며, $i, j, k$는 허수 단위로 다음 관계를 가진다:

$$i^2 = j^2 = k^2 = -1$$

$$ij = k, \quad jk = i, \quad ki = j$$

$$ji = -k, \quad kj = -i, \quad ik = -j$$

### 회전 쿼터니언

단위 쿼터니언($|\mathbf{q}| = 1$)으로 3D 회전을 표현한다:

$$\mathbf{q} = (\cos(\theta/2), \sin(\theta/2) \cdot \mathbf{u})$$

여기서 $\mathbf{u}$는 회전 축(단위 벡터), $\theta$는 회전 각도이다.

> **왜 4개 숫자인가?**  
> 3D 회전은 단위 회전축 $\mathbf{u}$와 회전각으로 결정된다. 단위 회전축 $\mathbf{u}$는 단위구 $S^2$ 위의 점이라 2자유도(위도·경도)이고, 회전각 1자유도를 더하면 회전 전체는 $SO(3)$ 3자유도다. 쿼터니언은 4성분이지만 단위 제약 $w^2+x^2+y^2+z^2=1$이 제약 1개를 걸어 $4-1=3$자유도가 정확히 채워진다. 오일러 각도는 3개 숫자만 쓰지만 짐벌락(특정 각도에서 자유도 하나가 소실되는 현상)이 생긴다. 쿼터니언은 4개 숫자를 사용하므로 짐벌락이 없고 모든 방향에서 균일하게 동작한다.

> **왜 θ/2(절반 각도)인가?**  
> 쿼터니언으로 벡터를 회전할 때는 샌드위치 곱 $\mathbf{v}' = \mathbf{q} \times \mathbf{v} \times \mathbf{q}^*$ 처럼 $\mathbf{q}$가 벡터의 양쪽에 두 번 곱해진다. 각 곱셈이 $\theta/2$씩 회전을 분담해 합쳐서 $\theta$만큼 회전한다. 엄밀히는 회전이 두 반사(reflection)의 합성인 것과 같은 원리다—각도 $\theta/2$로 놓인 두 거울에 차례로 비추면 두 거울 사이각의 2배인 $\theta$만큼 회전한다. $\mathbf{q}$가 한 번의 반사를, $\mathbf{q}^*$가 나머지 반사를 맡아 합쳐서 $\theta$ 회전을 만드는 셈이다. 또한 $\mathbf{q}$와 $-\mathbf{q}$가 같은 회전을 만들어내는 이중 덮개(double cover) 성질도 $\theta/2$ 표현 덕분에 자연스럽게 설명된다.

### 예시: Y축으로 90도 회전

$$\mathbf{u} = (0, 1, 0), \quad \theta = 90° = \pi/2$$

$$\mathbf{q} = (\cos(45°), 0, \sin(45°), 0) = (0.707, 0, 0.707, 0)$$

---

## 2. 쿼터니언의 구성 요소

### 실수부 (w / scalar part)

$$w = \cos(\theta/2)$$

회전각의 절반의 코사인이다. $w$가 1이면 회전이 0도(항등 회전)이고, $w$가 0이면 180도 회전이다.

### 허수부 (x, y, z / vector part)

$$(x, y, z) = \sin(\theta/2) \cdot \mathbf{u}$$

회전 축 방향 $\times$ 회전각 절반의 사인이다.

> **시계 비유**: $w = \cos(\theta/2)$는 회전각을 시계판 높이로 본 것이다. $w=1$은 12시(0°), $w=0$은 6시(180°), $w=-1$은 다시 12시(360°=0°)로, $|w|$가 클수록 회전각이 작다.

### 물리적 의미

$$|\mathbf{q}|^2 = w^2 + x^2 + y^2 + z^2 = 1 \quad \text{(단위 쿼터니언)}$$

**회전각 추출:**

$$\theta = 2 \cdot \arccos(w)$$

**회전축 추출:**

$$\mathbf{u} = \frac{(x, y, z)}{\sin(\theta/2)}$$

---

## 3. 쿼터니언 곱셈 (Hamilton Product)

두 쿼터니언의 곱은 회전의 합성이다.

### 공식

$$\mathbf{q}_1 \times \mathbf{q}_2 = (w_1 \cdot w_2 - \mathbf{v}_1 \cdot \mathbf{v}_2, \quad w_1 \cdot \mathbf{v}_2 + w_2 \cdot \mathbf{v}_1 + \mathbf{v}_1 \times \mathbf{v}_2)$$

전개하면:

$$q_w = w_1 \cdot w_2 - x_1 \cdot x_2 - y_1 \cdot y_2 - z_1 \cdot z_2$$

$$q_x = w_1 \cdot x_2 + x_1 \cdot w_2 + y_1 \cdot z_2 - z_1 \cdot y_2$$

$$q_y = w_1 \cdot y_2 - x_1 \cdot z_2 + y_1 \cdot w_2 + z_1 \cdot x_2$$

$$q_z = w_1 \cdot z_2 + x_1 \cdot y_2 - y_1 \cdot x_2 + z_1 \cdot w_2$$

### 회전 합성

$$\text{combined} = \mathbf{q}_2 \times \mathbf{q}_1 \quad (\mathbf{q}_1 \text{ 회전 후 } \mathbf{q}_2 \text{ 회전})$$

이 규칙은 **외인적(extrinsic, 고정 월드 축)** 해석이다. 외인적(월드 축 고정)은 곱하는 순서가 적용 순서와 같고, 내인적(intrinsic, 로컬 축)은 반대다.

> **주의**: 행렬과 마찬가지로 순서가 중요하다! $\mathbf{q}_1 \times \mathbf{q}_2 \neq \mathbf{q}_2 \times \mathbf{q}_1$

**게임에서의 활용:**
```text
// 월드 축(고정) 기준으로 pitch(Y) → yaw(Z) 순서로 회전 합성
q_pitch = fromAxisAngle((0,1,0), 30°)    // pitch = Y축
q_yaw = fromAxisAngle((0,0,1), 90°)      // yaw = Z축
final = q_yaw × q_pitch    // pitch(Y) 먼저 적용, yaw(Z) 나중 — extrinsic
```

---

## 4. 켤레 (Conjugate)와 역 (Inverse)

### 켤레 쿼터니언

$$\mathbf{q}^* = (w, -x, -y, -z) \quad \text{(벡터부 부호 반전)}$$

### 역 쿼터니언

$$\mathbf{q}^{-1} = \frac{\mathbf{q}^*}{|\mathbf{q}|^2}$$

단위 쿼터니언의 경우 ($|\mathbf{q}| = 1$):

$$\mathbf{q}^{-1} = \mathbf{q}^* \quad \text{(켤레 = 역)}$$

### 의미

$$\mathbf{q} \times \mathbf{q}^{-1} = 1 \quad \text{(항등 쿼터니언 } (1,0,0,0) \text{)}$$

역 쿼터니언은 반대 방향 회전이다:

$$\mathbf{q} = (\cos(\theta/2), \sin(\theta/2) \cdot \mathbf{u})$$

$$\mathbf{q}^{-1} = (\cos(\theta/2), -\sin(\theta/2) \cdot \mathbf{u}) \quad \text{($-\theta$ 회전)}$$

---

## 5. 노름 (Norm)

$$|\mathbf{q}| = \sqrt{w^2 + x^2 + y^2 + z^2}$$

### 단위 쿼터니언

회전을 표현하려면 $|\mathbf{q}| = 1$이어야 한다. 보통 연산 후 부동소수점 오차로 노름이 1에서 벗어나므로 정규화가 필요하다.

$$\mathbf{q} = \frac{\mathbf{q}}{|\mathbf{q}|} \quad \text{(정규화)}$$

> **실전 팁**: 쿼터니언을 사용한 회전 연산 후에는 항상 정규화하자. 누적 오차가 회전을 왜곡할 수 있다.

---

## 6. 회전 적용 (Vector Rotation)

쿼터니언 $\mathbf{q}$로 벡터 $\mathbf{v}$를 회전:

### 공식 (Sandwich Product)

$$\mathbf{v}' = \mathbf{q} \times \mathbf{v} \times \mathbf{q}^{-1}$$

$\mathbf{v}$를 순수 쿼터니언 $(0, \mathbf{v})$로 취급한다. $\mathbf{q}$가 단위 쿼터니언이면 $\mathbf{q}^{-1} = \mathbf{q}^*$이다.

### 최적화된 버전

직접 회전 (곱셈 2번 대신 더 빠른 방법):

$$\mathbf{t} = 2 \cdot (\mathbf{q}_{xyz} \times \mathbf{v})$$

$$\mathbf{v}' = \mathbf{v} + q_w \cdot \mathbf{t} + (\mathbf{q}_{xyz} \times \mathbf{t})$$

**게임에서의 활용:**
```text
// 캐릭터의 forward 벡터 회전
forward = q × (0, 0, -1) × q*
// 또는
forward = rotate(q, (0, 0, -1))
```

> **관례**: 이 문서는 $-Z$ forward 관례(《LookAt》과 동일)를 쓴다. 《회전 & 보간》 §8은 $+Z$ forward를 가정하므로 코드 예시의 부호가 다르다.

---

## 7. 축-각도 (Axis-Angle) 변환

### 축-각도 → 쿼터니언

$$\mathbf{q} = (\cos(\theta/2), \quad \mathbf{u}_x \cdot \sin(\theta/2), \quad \mathbf{u}_y \cdot \sin(\theta/2), \quad \mathbf{u}_z \cdot \sin(\theta/2))$$

```text
function fromAxisAngle(axis, angle):
    axis = normalize(axis)
    half = angle / 2
    s = sin(half)
    return (cos(half), axis.x × s, axis.y × s, axis.z × s)
```

### 쿼터니언 → 축-각도

$$\theta = 2 \cdot \arccos(w)$$

$$s = \sqrt{1 - w^2}$$

$s < 0.001$이면 회전이 거의 없으므로 임의의 축 $(1, 0, 0)$을 사용하고, 그렇지 않으면:

$$\mathbf{u} = \frac{(x, y, z)}{s}$$

```text
function toAxisAngle(q):
    q = normalize(q)
    angle = 2 × acos(w)
    s = sqrt(1 - w²)
    if s < 0.001:
        axis = (1, 0, 0)    // 임의의 축 (회전이 거의 없음)
    else:
        axis = (x/s, y/s, z/s)
    return axis, angle
```

---

## 8. 쿼터니언 vs 오일러 각도 vs 행렬

| 특징 | 오일러 각도 | 행렬 (3×3) | 쿼터니언 |
|------|-----------|-----------|---------|
| 저장 | 3 float | 9 float | 4 float |
| 짐벌락 | 있음 | 없음 | 없음 |
| 보간 | 어려움 | 불가능 | Slerp |
| 회전 합성 | 어려움 | 곱셈 | 곱셈 |
| 회전 적용 | 직관적 | 행렬×벡터 | sandwich |
| 직관성 | 높음 | 낮음 | 중간 |
| 보간 품질 | 나쁨 | N/A | 우수 |

> **실전 추천**: 내부 계산은 쿼터니언, 에디터/UX 표시는 오일러 각도, 셰이더/버퍼 전달은 행렬로 변환.

---

## 9. 실전 예시: 캐릭터 회전

```text
// 키보드 입력으로 캐릭터 회전
yaw += inputX × turnSpeed × dt
pitch += inputY × turnSpeed × dt
pitch = clamp(pitch, -89°, 89°)

// 오일러 → 쿼터니언
q = fromEuler(pitch, yaw, roll)

// 캐릭터 모델에 적용
character.rotation = q

// forward 벡터 추출
forward = rotate(q, (0, 0, -1))
```

> **관례**: $-Z$ forward 관례 (위 §6 참고). 《회전 & 보간》 §8은 $+Z$ forward를 가정한다.

---

## 10. 빠른 참조

| 연산 | 공식 |
|------|------|
| 정의 | $\mathbf{q} = (\cos(\theta/2), \sin(\theta/2) \cdot \mathbf{u})$ |
| 노름 | $|\mathbf{q}| = \sqrt{w^2 + x^2 + y^2 + z^2}$ |
| 곱셈 | $\mathbf{q}_1 \times \mathbf{q}_2 = (w_1 w_2 - \mathbf{v}_1 \cdot \mathbf{v}_2, \; w_1 \mathbf{v}_2 + w_2 \mathbf{v}_1 + \mathbf{v}_1 \times \mathbf{v}_2)$ |
| 켤레 | $\mathbf{q}^* = (w, -x, -y, -z)$ |
| 역 | $\mathbf{q}^{-1} = \mathbf{q}^*$ (단위 쿼터니언) |
| 회전 적용 | $\mathbf{v}' = \mathbf{q} \times \mathbf{v} \times \mathbf{q}^{-1}$ |
| 회전각 | $\theta = 2 \cdot \arccos(w)$ |
| 회전추출 | $\mathbf{u} = (x,y,z) / \sin(\theta/2)$ |