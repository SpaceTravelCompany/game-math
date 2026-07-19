---
title: 삼각함수 활용 (Trigonometry Applications)
slug: trig-applications
---

## 소개

삼각함수는 게임에서 단순한 각도 계산을 넘어, 원운동, 파동 효과, 탄도, 스무스 스텝, 방향 계산 등 다양한 곳에 활용된다. 이 페이지에서는 실전에서 바로 쓸 수 있는 응용 기법을 다룬다.

> **핵심**
> atan2로 방향을 구하고, sin/cos로 파동과 원운동을 만들고, smoothstep으로 부드러운 전환을 한다.

---

## 1. Atan2 — 방향 각도 구하기

게임에서 가장 자주 쓰는 삼각함수 응용이다.

### 타겟을 향한 각도

객체 $A$가 $B$를 바라보는 각도:

$$dx = B_x - A_x$$

$$dy = B_y - A_y$$

$$\theta = \operatorname{atan2}(dy,\, dx)$$

이 각도로 이동:

$$v_x = \text{speed} \times \cos\theta$$

$$v_y = \text{speed} \times \sin\theta$$

### 3D에서의 방향

Y축(수직축) 주위 회전 (yaw), $dx$, $dz$는 수평면 성분:

$$\text{yaw} = \operatorname{atan2}(dx,\, dz)$$

X축(수직) 회전 (pitch):

$$\text{horizontalDist} = \sqrt{dx^2 + dz^2}$$

$$\text{pitch} = \operatorname{atan2}(dy,\, \text{horizontalDist})$$

**게임에서의 활용:**
- **유도 미사일**: 타겟을 향해 방향 회전
- **타워 디펜스**: 타워가 적을 향해 회전
- **캐릭터 방향**: 이동 방향에 맞춰 스프라이트 회전

```text
// 유도 미사일
angle = atan2(target.y - missile.y, target.x - missile.x)
missile.vx = speed × cos(angle)
missile.vy = speed × sin(angle)
```

---

## 2. 파동 효과 (Wave)

sin/cos을 시간에 적용하면 주기적인 파동을 만든다.

### 기본 파동

$$y = A \times \sin(\omega \times t + \phi)$$

- $A$: 진폭 (높이)
- $\omega$: 각주파수 (angular frequency, rad/s; $\omega = 2\pi f$)
- $\phi$: 위상 (시작 위치)

### 게임 활용

```text
// 떠다니는 아이템
item.y = baseY + 0.3 × sin(time × 2)

// 호흡 효과 (스케일)
scale = 1 + 0.05 × sin(time × 3)

// 깜빡이는 효과
opacity = 0.5 + 0.5 × sin(time × 5)
```

### 다중 파동 합성

$$\text{wave}_1 = 0.5 \times \sin(t \times 1.0 + x \times 0.3)$$

$$\text{wave}_2 = 0.3 \times \sin(t \times 2.3 + x \times 0.5)$$

$$\text{wave}_3 = 0.2 \times \sin(t \times 5.1 + x \times 1.2)$$

$$\text{height} = \text{wave}_1 + \text{wave}_2 + \text{wave}_3$$

> **실전 팁**: 자연스러운 파동은 단일 주파수가 아니라 여러 주파수의 합으로 만들어진다. 푸리에 급수의 아이디어를 차용한 것.

---

## 3. 원운동 (Circular Motion)

### 2D 원운동

$$\theta = t \times \omega$$

$$x = c_x + r \times \cos\theta$$

$$y = c_y + r \times \sin\theta$$

### 3D 원운동 (궤도)

$$\theta = t \times \text{orbitSpeed}$$

$$x = c_x + r \times \cos\theta$$

$$z = c_z + r \times \sin\theta$$

$$y = c_y + \text{heightOffset}$$

### 타원 궤도

$$x = c_x + a \times \cos\theta$$

$$y = c_y + b \times \sin\theta$$

**게임에서의 활용:**
- **궤도 카메라**: 플레이어 주위를 도는 카메라
- **회전하는 위험 요소**: 톱, 회전하는 가시, 레이저
- **군집 이동**: 한 점 주위를 회전하는 무리

```text
// 회전하는 가시 (spinning trap)
for i in range(numSpikes):
    angle = time × spinSpeed + i × (2π / numSpikes)
    spike.x = center.x + radius × cos(angle)
    spike.y = center.y + radius × sin(angle)
```

---

## 4. 진자 (Pendulum)

$$\theta = A \times \cos\left(\sqrt{\frac{g}{L}} \times t\right)$$

> **적용 조건**: 이 공식은 작은 각도 근사($\sin\theta \approx \theta$, 예 $10°$ 이하)에서만 성립한다. 큰 각도 진자는 주기가 진폭에 의존하므로 이 공식에서 벗어난다.

- $g$: 중력가속도
- $L$: 진자 길이
- $A$: 최대 각도 (라디안)

**게임에서의 활용:**
- **스윙**: 줄에 매달린 캐릭터
- **매달린 물체**: 램프, 깃발, 사슬
- **무기**: 도끼, 망치 회전

---

## 5. 탄도 (Projectile Motion)

중력을 받는 발사체의 궤적을 계산한다.

### 기본 탄도

$$x(t) = x_0 + v \times \cos\theta \times t$$

$$y(t) = y_0 + v \times \sin\theta \times t - \frac{1}{2}g \times t^2$$

최고 도달 시간:

$$t_{\text{peak}} = \frac{v \times \sin\theta}{g}$$

최대 높이:

$$h_{\text{max}} = \frac{(v \times \sin\theta)^2}{2g}$$

사거리 (수평 거리):

$$\text{range} = \frac{v^2 \times \sin(2\theta)}{g}$$

### 45도가 최대 사거리인 이유

$\sin(2\theta)$가 $\theta = 45°$에서 최대값 1이 된다. 즉, 같은 초기 속도면 45도가 가장 멀리 간다.

$$\text{range}_{\text{max}} = \frac{v^2}{g} \quad (\theta = 45°)$$

**게임에서의 활용:**
- **포탄**: 대포, 투석기
- **점수**: 공중으로 던지는 아이템
- **예측**: 발사체가 어디에 떨어질지 표시

```text
// 발사각도 계산 (타겟이 알려진 경우)
function launchAngle(origin, target, speed, gravity):
    dx = target.x - origin.x
    dy = target.y - origin.y

    // 2차 방정식의 해
    g = gravity
    v = speed
    v2 = v × v
    v4 = v2 × v2

    discriminant = v4 - g × (g × dx² + 2 × dy × v2)

    if discriminant < 0:
        return null    // 도달 불가

    sqrtD = sqrt(discriminant)
    tanθ1 = (v2 + sqrtD) / (g × dx)
    tanθ2 = (v2 - sqrtD) / (g × dx)

    // 더 낮은 궤적(tanθ2)을 보통 선택
    return atan(tanθ2)
```

---

## 6. Smoothstep

부드러운 보간 함수로, 0에서 1로 부드럽게 전환한다.

### 공식

$$\text{smoothstep}(t) = t^2 \times (3 - 2t)$$

### 3-인수 일반형 (GLSL·HLSL `smoothstep`)

셰이더의 `smoothstep(edge0, edge1, x)`는 입력을 $[e_0, e_1]$ 구간에서 $[0, 1]$로 clamp·정규화한 뒤 위 공식을 적용한다:

$$\text{smoothstep}(e_0,e_1,x)=3t^2-2t^3,\quad t=\text{clamp}\!\left(\frac{x-e_0}{e_1-e_0},\,0,\,1\right)$$

> **cubic Hermite 연결**: 이 $3t^2 - 2t^3$은 cubic Hermite 스플라인의 Hermite 기저 중 **도착점 계수 $h_{01}(t) = 3t^2 - 2t^3$** 이다. 출발점 계수는 $h_{00}(t) = 2t^3 - 3t^2 + 1 = 1 - \text{smoothstep}(t)$이며, 양 끝 접선이 $0$인 단위 보간을 만든다.

### 더 부드러운 버전 (Smootherstep)

$$\text{smootherstep}(t) = t^3 \times (t \times (6t - 15) + 10)$$

### 특징

$$\text{smoothstep}(0) = 0 \quad \text{(시작)}$$

$$\text{smoothstep}(1) = 1 \quad \text{(끝)}$$

$$\text{smoothstep}(0.5) = 0.5 \quad \text{(중간)}$$

미분이 0인 지점: $t = 0$, $t = 1$ (끝에서 부드럽게 멈춤)

**게임에서의 활용:**
- **색상 전환**: 낮/밤 전환
- **높이 안개**: 특정 높이에서 안개 시작/종료
- **카메라 이동**: 시작과 끝이 부드러운 이동
- **셰이더**: 텍스처 혼합 경계 부드럽게

```text
// 낮/밤 전환
nightAmount = smoothstep(sunsetStart, sunsetEnd, time)
skyColor = lerp(dayColor, nightColor, nightAmount)

// 높이 안개
fogAmount = smoothstep(fogStart, fogEnd, height)
```

> **참고**: Easing·스칼라 보간 활용은 《Easing 함수》, 프레임 독립 보간은 《벡터 보간》을 참고.

---

## 7. Lissajous 곡선

두 수직 파동의 조합으로 복잡한 패턴을 만든다.

$$x = A \times \sin(a \times t + \delta)$$

$$y = B \times \sin(b \times t)$$

- $A$, $B$: 진폭
- $a$, $b$: 주파수 비율
- $\delta$: 위상차 — 한 축에만 적용. 위쪽 정의는 x축에 놓지만, 아래 코드처럼 y축에 넣어도 같은 Lissajous 족이다

**게임에서의 활용:**
- **파티클 패턴**: 화려한 효과 궤적
- **HUD 애니메이션**: 메뉴 항목의 부동 운동
- **보스 패턴**: 복잡한 이동 패턴

```text
// 보스의 Lissajous 패턴 (위상차 δ=π/2를 y축에 적용)
x = centerX + 200 × sin(0.5 × time)
y = centerY + 150 × sin(0.7 × time + π/2)
```

---

## 8. 각도 보간

### 선형 각도 보간 (주의)

```text
// 단순 lerp는 350° → 10°를 340도 회전함 (틀림)
angle = lerp(350°, 10°, t)  // → 340도 회전 (실제로는 20도여야 함)
```

올바른 방법: 최단 각도 차이를 구한다.

라디안 기준:

$$\text{diff} = ((b - a + \pi) \bmod 2\pi) - \pi$$

도(degree) 단위라면 $2\pi \to 360$, $\pi \to 180$으로 바꾼다:

$$\text{diff} = ((b - a + 180) \bmod 360) - 180$$

$$\text{result} = a + \text{diff} \times t$$

### Slerp 적용 (3D)

```text
// 쿼터니언 Slerp 사용 (각도 보간의 3D 버전)
rotation = slerp(currentRot, targetRot, t)
```

---

## 9. 빠른 참조

| 활용 | 공식 | 게임 예시 |
|------|------|----------|
| 방향 각도 | $\operatorname{atan2}(dy,\, dx)$ | 유도 미사일 |
| 파동 | $A \times \sin(\omega t + \phi)$ | 떠다니는 아이템 |
| 원운동 | $(r \times \cos\theta,\, r \times \sin\theta)$ | 회전 가시, 궤도 |
| 진자 | $A \times \cos\!\left(\sqrt{g/L} \times t\right)$ | 스윙 |
| 탄도 | $v \times \cos\theta \times t,\; v \times \sin\theta \times t - \frac{1}{2}gt^2$ | 포탄 |
| Smoothstep | $t^2 \times (3 - 2t)$ | 색상 전환 |
| 각도 보간 | $((b - a + \pi) \bmod 2\pi) - \pi$ | 부드러운 회전 |
| Lissajous | $\sin(at),\, \sin(bt)$ | 패턴 이동 |