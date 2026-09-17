---
title: 스프링 & 댐핑 (Spring & Damping)
slug: spring-damping
---

## 소개

스프링-댐퍼 시스템은 물리 기반의 부드러운 복원 운동을 구현하는 2차 동역학(second-order dynamics) 모델이다. 게임에서 무기 흔들림 복원, 카메라 충격 반응, 탄성 오브젝트, 캐릭터 서스펜션 등 "튕기면서 원래 자리로 돌아오는" 동작이 필요할 때 사용된다.

> **핵심**
> 스프링은 목표를 향해 힘을 가하지만 관성 때문에 오버슈트가 발생할 수 있다. 댐핑은 그 진동을 제어한다.

> **참고**: 단조 수렴(overshoot 없음)이 필요하면 스프링 대신 《지수 감쇠》 또는 《벡터 보간》의 SmoothDamp를 사용한다.

---

## 1. 훅의 법칙 (Hooke's Law)

스프링이 평형 위치에서 벗어났을 때 원래대로 돌아가려는 복원력은 변위에 비례한다:

$$F_{\text{spring}} = -k\,x$$

- $x$: 평형 위치로부터의 변위 (벡터 또는 스칼라)
- $k$: 스프링 강성(spring stiffness). 클수록 빠르게 복원
- 음의 부호 $-$: 항상 평형 방향으로 힘이 작용

**게임에서의 활용:**
- **무기 흔들림**: 조준점이 밀려난 후 원래 위치로 복원
- **탄성 로프**: 양 끝이 당겨지면 복원력 발생

```text
// 1D 스프링 힘
float displacement = currentPos - restPos;
float springForce = -k * displacement;
```

---

## 2. 운동 방정식 (Equation of Motion)

스프링 힘만 있으면 영원히 진동한다. 여기에 **댐핑**(속도에 비례하는 저항)을 추가한다:

$$m\,a = -k\,x - c\,v$$

- $m$: 질량 (관성)
- $a$: 가속도 (위치의 2차 미분)
- $c$: 댐핑 계수 (damping coefficient)
- $v$: 속도 (위치의 1차 미분)

### 댐핑의 세 가지 영역

임계 댐핑 계수:

$$c_{\text{crit}} = 2\sqrt{k m}$$

이를 정규화한 **감쇠비**(damping ratio) $\zeta$가 업계 표준 튜닝 언어다:

$$\zeta = \frac{c}{2\sqrt{k m}} = \frac{c}{c_{\text{crit}}}$$

$\zeta < 1$이면 저감쇠(진동), $\zeta = 1$이면 임계 감쇠(가장 빠른 무진동 수렴), $\zeta > 1$이면 과감쇠(느린 수렴)다.

| 상태 | 조건 | 동작 |
|------|------|------|
| **과댐핑** (Overdamped) | $c > c_{\text{crit}}$ | 천천히 수렴, 진동 없음 |
| **임계 댐핑** (Critically damped) | $c = c_{\text{crit}}$ | 가장 빠르게 수렴, 진동 없음 |
| **저댐핑** (Underdamped) | $c < c_{\text{crit}}$ | 진동하며 수렴, 오버슈트 발생 |

```text
    위치
    ↑
    │    저댐핑 ~~~~~~→ (진동하며 수렴)
    │    임계 ──────→ (가장 빠름, 진동 없음)
    │    과댐 ─────────→ (느리게 수렴)
    └──────────────────→ 시간
```

### 실무 튜닝 팁: $\omega_0$와 $\zeta$로 한 번에 세팅하기

스프링($k$), 댐퍼($c$), 질량($m$) 3개 숫자를 무작정 찍어서 맞추기는 매우 어렵다.  
현업에서는 **질량을 $m=1$로 고정**하고, 감각적인 두 값으로 자동 계산한다:

1. **복원 속도 ($\omega_0$)**: 얼마나 빠르게 제자리로 올 것인가? (보통 $5 \sim 25$)
2. **튕김 정도 ($\zeta$, 감쇠비)**: 얼마나 통통 튈 것인가?
   - $\zeta = 1.0$: **임계 감쇠** — 튕김 전혀 없이 가장 빠르게 착 멈춤 (카메라, 조준선)
   - $\zeta = 0.5 \sim 0.7$: **자연스러운 탄성** — 한두 번 통 튕기고 안정화 (피격 흔들림, UI)
   - $\zeta = 0.2 \sim 0.3$: **말랑한 젤리** — 여러 번 통통통 튕김 (슬라임, 탄성 오브젝트)

$$k = \omega_0^2, \qquad c = 2 \cdot \zeta \cdot \omega_0$$

```text
// 예: 빠른 복원(ω0 = 15), 자연스러운 탄성(ζ = 0.6)
float omega = 15.0f;
float zeta = 0.6f;
float k = omega * omega;          // 225.0
float c = 2.0f * zeta * omega;    // 18.0
float m = 1.0f;
```

---

## 3. 수치 적분 (Numerical Integration)

매 프레임마다 운동 방정식을 수치 적분한다. **Semi-implicit Euler**(심플렉틱 오일러)가 게임에서 가장 널리 쓰인다:

```text
// 매 프레임 호출
void updateSpring(float& x, float& v, float dt) {
    float force = -k * x - c * v;  // 복원력 + 댐핑
    float a = force / m;           // 가속도 (F=ma)
    v += a * dt;                   // 속도 갱신 (semi-implicit)
    x += v * dt;                   // 위치 갱신
}
```

**왜 Semi-implicit Euler인가:**
- Explicit Euler(먼저 $x$ 갱신 후 $v$)보다 에너지 보존이 우수
- 구현이 단순하고 안정적
- $dt$가 너무 크면(FPS 낮음) 발산 가능 → 프레임 상한 또는 sub-step 필요

```text
// 안정성을 위한 sub-step 분할
void updateSpring(float& x, float& v, float dt) {
    // dt=0이면 steps=0 → subDt=0/0=NaN. max(1, ...)로 최소 1보장
    int steps = max(1, (int)ceil(dt / maxDt));  // maxDt = 1/60 정도
    float subDt = dt / steps;
    for (int i = 0; i < steps; i++) {
        float force = -k * x - c * v;
        v += (force / m) * subDt;
        x += v * subDt;
    }
}
```

---

## 4. 게임 활용

### 무기 흔들림 복원 (Weapon Bob Recovery)

FPS에서 조준/이동 후 총구를 원위치로 복원:

```text
// 총기 반동 복원 (임계 댐핑에 가깝게)
k = 40.0f;    // 강한 복원
c = 12.0f;    // 임계에 가까운 댐핑 (2*sqrt(40*1) ≈ 12.65)
m = 1.0f;

updateSpring(gunOffset.x, gunVel.x, dt);
updateSpring(gunOffset.y, gunVel.y, dt);
```

### 카메라 충격 복원

폭발/피격 시 카메라가 밀려났다가 돌아오기:

```text
// 카메라 셰이크 복원 (저댐핑 — 약간의 진동으로 충격감)
k = 25.0f;
c = 3.0f;     // 저댐핑 (c < 2*sqrt(25*1) = 10)
m = 1.0f;
```

### 젤리/탄성 오브젝트

적을 타격했을 때 일시적으로 스케일이 찌그러졌다 복원:

```text
// 찌그러짐 복원 (저댐핑, 튕김 있게)
k = 30.0f;
c = 2.0f;
m = 1.0f;
```

### 스냅백 (Snap-Back)

낚시 줄, grapple hook, 밧줄이 당겨졌다가 복원:

```text
// 밧줄 복원: 변위가 ropeLength를 초과하면 스프링 작용
float displacement = length(currentPos - anchor) - ropeLength;
if (displacement > 0) {
    float force = -k * displacement;
    // ...
}
```

---

## 5. 스프링 vs Lerp / SmoothDamp

| 특징 | 스프링 (Spring) | Lerp (지수 감쇠) | SmoothDamp |
|------|----------------|-------------------|------------|
| **차수** | 2차 동역학 | 1차 동역학 | 2차 (임계 댐핑) |
| **오버슈트** | 가능 (저댐핑 시) | 없음 | 거의 없음 |
| **진동** | 가능 | 없음 | 없음 |
| **관성** | 있음 (질량) | 없음 | 있음 (속도 추적) |
| **튜닝 파라미터** | $k, c, m$ 3개 | $r$ (rate) 1개 | smoothTime 1개 |
| **계산 비용** | 중간 | 낮음 | 중간 |

### 언제 무엇을 쓸까

- **오버슈트가 필요하다면** → 스프링 (저댐핑). 젤리, 탄성 효과.
- **가장 빠르게 안정화** → 스프링 (임계 댐핑). 또는 SmoothDamp.
- **튜닝을 간단하게** → Lerp/지수 감쇠. rate 하나로 끝.
- **목표값이 급변하는 환경** → 스프링. 관성이 급변을 자연스럽게 걸러줌.
- **직관적인 복원 시간 제어** → SmoothDamp. `smoothTime`으로 직접 지정.

> **실전 팁**: 3개의 파라미터($k$, $c$, $m$)를 모두 튜닝하는 것은 까다롭다. 보통 $m=1$로 고정하고 $k$로 복원 속도, $c$로 댐핑 정도만 조정한다. 혹은 SmoothDamp(임계 댐핑 근사)로 시작해 필요시 스프링으로 대체하는 것이 일반적이다.

---

## 6. 빠른 참조

| 개념 | 수식 | 설명 |
|------|------|------|
| 훅의 법칙 | $F = -k\,x$ | 복원력 = 강성 × 변위 |
| 운동 방정식 | $m\,a = -k\,x - c\,v$ | 질량 × 가속도 = 복원력 + 댐핑 |
| 임계 댐핑 | $c_{\text{crit}} = 2\sqrt{km}$ | 진동 없는 최소 댐핑 |
| 감쇠 포락선 (진폭 감쇠 비율) | $e^{-c\,t/(2m)}$ | 진폭 감쇠 비율 |
| 감쇠비 | $\zeta = \dfrac{c}{2\sqrt{km}}$ | $\zeta<1$ 저감쇠, $=1$ 임계, $>1$ 과감쇠 |
| 비감쇠 고유진동수·주기 | $\omega_0 = \sqrt{k/m}$, $T = 2\pi/\omega_0$ | 감쇠 없는 자연 진동수와 주기 |
| 진폭 반감기 | $t_{1/2} = \dfrac{2m\ln 2}{c}$ | 진폭이 절반으로 줄어드는 시간 |
| 2% 정착시간 | $\approx \dfrac{8m}{c} = \dfrac{4}{\zeta\omega_0}$ | 진폭이 2% 이내로 수렴 ($4m/c$는 잔차 13.5%) |
| 각진동수 (저댐핑) | $\omega_d = \sqrt{\frac{k}{m} - \frac{c^2}{4m^2}}$ | 감쇠 진동의 주파수 |

| 용도 | 추천 댐핑 | $k$ (m=1 기준) |
|------|----------|---------------|
| 무기 복원 | 임계 ~ 약임계 | 30~50 |
| 카메라 충격 | 저댐핑 | 20~30 |
| 젤리/탄성 | 저댐핑 (튕김) | 25~40 |
| 스냅백 (밧줄) | 과댐핑 | 50~100 |
| 캐릭터 서스펜션 | 임계 | 40~60 |
| 메뉴 애니메이션 | SmoothDamp 권장 | — |
