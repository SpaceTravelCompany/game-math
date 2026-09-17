---
title: 충돌처리 (Collision Response)
slug: collision-response
---

## 소개

《충돌 검출》(collision-detection)이 "두 객체가 겹치는가"를 판별하는 문제라면, **충돌처리(Response)**는 "겹쳤을 때 어떻게 밀어내고, 튕기고, 멈출 것인가"를 다룬다. 물리 엔진에서 검출이 절반이라면 나머지 절반이 응답이다.

> **핵심**
> 충돌 응답의 세 축: **위치 보정**(관통 해소), **속도 변화**(반사/멈춤), **마찰**(접선 감쇠).

> **참고**: 이 문서는 물리 응답에 집중한다. 검출 알고리즘 자체는 《충돌 검출》, 벽 슬라이딩은 《거리 & 최단점》 §9, 응용 사례는 《플랫포머 물리》《슈팅 게임 물리》를 각각 참고하라.

---

## 1. 검출 → 응답 파이프라인

물리 엔진의 매 프레임 흐름:

```text
1. Broad Phase (브로드 페이즈)
   → AABB/구로 충돌 가능 쌍 추리기

2. Narrow Phase (내로우 페이즈)
   → SAT, GJK, Sphere 등 정밀 검사
   → 충돌 법선(n), 관통 깊이(depth) 출력

3. Collision Response (충돌 응답)
   → 임펄스로 속도 변경 (반발, 마찰)
   → 위치 보정으로 관통 해소
   → 접촉점 기준 토크 적용 (회전)
```

---

## 2. 충돌 법선과 관통 깊이

검출 단계에서 얻은 두 값이 응답의 입력이 된다:

- **충돌 법선** $\mathbf{n}$: B를 기준으로 A를 밀어낼 방향의 단위 벡터
- **관통 깊이** $d$: 두 객체가 겹친 정도 (최소 분리 거리)

> 자세한 계산법(MTV, SAT, GJK+EPA)은 《충돌 검출》 §5(SAT·MTV), §6~§7(GJK·EPA) 참고.

### MTV (Minimum Translation Vector)

$$\text{MTV} = \mathbf{n} \cdot d$$

이 벡터를 따라 객체를 밀어내면 관통이 해소된다. 어떤 객체를 얼마나 밀지는 질량 비율로 결정한다.

---

## 3. 임펄스 기반 응답 (Impulse)

### 선운동량 보존과 반발 계수

충돌 직전 두 물체의 상대 속도:

$$\mathbf{v}_{rel} = \mathbf{v}_A - \mathbf{v}_B$$

법선 방향의 상대 속도:

$$v_n = \mathbf{v}_{rel} \cdot \mathbf{n}$$

**반발 계수**(Coefficient of Restitution) $e$는 충돌 후 속도의 반사 정도를 결정한다:

- $e = 0$: 완전 비탄성 (충돌 후 붙음)
- $e = 1$: 완전 탄성 (에너지 손실 없음)
- $0 < e < 1$: 현실적 반발

### 임펄스 크기

$$j = \frac{-(1 + e)\, v_n}{\displaystyle\frac{1}{m_A} + \frac{1}{m_B}}$$

### 속도 갱신

$$\mathbf{v}_A' = \mathbf{v}_A + \frac{j}{m_A}\,\mathbf{n}$$

$$\mathbf{v}_B' = \mathbf{v}_B - \frac{j}{m_B}\,\mathbf{n}$$

```text
// 충돌 응답 (회전 미포함)
void resolveCollision(
    float& vx_A, float& vy_A, float massA,
    float& vx_B, float& vy_B, float massB,
    float nx, float ny, float restitution
) {
    // 법선 방향 상대 속도
    float relVn = (vx_A - vx_B) * nx + (vy_A - vy_B) * ny;

    if (relVn > 0) return;  // 이미 분리 중 → 무시

    float invMassSum = 1.0f / massA + 1.0f / massB;
    float j = -(1 + restitution) * relVn / invMassSum;

    vx_A += (j / massA) * nx;
    vy_A += (j / massA) * ny;
    vx_B -= (j / massB) * nx;
    vy_B -= (j / massB) * ny;
}
```

### 정적 물체 (무한 질량)

바닥·벽처럼 움직이지 않는 정적 물체는 **역질량 $\text{invMass} = 1/m = 0$** 으로 모델링한다. 임펄스·위치 보정 공식의 $1/m$ 자리에 그대로 0을 대입하면 된다 — 정적 물체는 속도·위치 변화가 0이고, 분모는 $1/m_A + 0 = 1/m_A$로 줄어 수치적으로 안정하다(질량 $\infty$를 직접 다루지 않아도 된다).

```text
// invMass 기반 충돌 응답 (정적 물체는 invMass = 0)
struct Body { float invMass; float vx, vy; ... };  // 정적: invMass = 0

float invMassSum = a.invMass + b.invMass;
if (invMassSum == 0.0f) return;      // 둘 다 정적이면 응답 없음

float j = -(1 + restitution) * relVn / invMassSum;

// 정적 물체는 invMass = 0이라 속도 변화가 자동으로 0
a.vx += (j * a.invMass) * nx;
a.vy += (j * a.invMass) * ny;
b.vx -= (j * b.invMass) * nx;
b.vy -= (j * b.invMass) * ny;
```

### 회전이 포함된 경우

접촉점에서의 속도는 회전 속도까지 포함한다. 접촉점 기준 $\mathbf{r}$은 질량 중심에서 접촉점까지의 벡터:

$$\mathbf{v}_{\text{atPoint}} = \mathbf{v}_{\text{COM}} + \boldsymbol{\omega} \times \mathbf{r}$$

따라서 접촉점에서의 상대 법선 속도는 양쪽 물체의 선운동량·각운동량을 모두 반영한다:

$$v_n = (\mathbf{v}_A + \boldsymbol{\omega}_A \times \mathbf{r}_A - \mathbf{v}_B - \boldsymbol{\omega}_B \times \mathbf{r}_B) \cdot \mathbf{n}$$

임펄스 크기의 분모에는 회전 관성의 결합 항이 더해진다 (Box2D/Randy Gaul 표준 형식):

$$j=\frac{-(1+e)\,v_n}{\dfrac1{m_A}+\dfrac1{m_B}+\bigl((I_A^{-1}(\mathbf r_A\times\mathbf n))\times\mathbf r_A\bigr)\cdot\mathbf n+\bigl((I_B^{-1}(\mathbf r_B\times\mathbf n))\times\mathbf r_B\bigr)\cdot\mathbf n}$$

임펄스 $j$가 접촉점에 가해지면 선운동량과 각운동량이 함께 변한다:

$$\mathbf{v}_A' = \mathbf{v}_A + \frac{j}{m_A}\,\mathbf{n}, \qquad \mathbf{v}_B' = \mathbf{v}_B - \frac{j}{m_B}\,\mathbf{n}$$

$$\boldsymbol\omega_A'=\boldsymbol\omega_A+I_A^{-1}(\mathbf r_A\times(j\mathbf n)),\qquad \boldsymbol\omega_B'=\boldsymbol\omega_B-I_B^{-1}(\mathbf r_B\times(j\mathbf n))$$

---

## 4. 마찰 (Friction)

충돌 시 접선 방향의 속도도 처리해야 자연스럽다.

### 접선 임펄스

법선 방향과 직교하는 접선 속도:

$$\mathbf{v}_t = \mathbf{v}_{rel} - (\mathbf{v}_{rel} \cdot \mathbf{n})\,\mathbf{n}$$

접선 방향 단위 벡터:

$$\mathbf{t} = \frac{\mathbf{v}_t}{|\mathbf{v}_t|}$$

### Coulomb 마찰 한계

$$j_t = -\frac{\mathbf{v}_{rel} \cdot \mathbf{t}}{\displaystyle\frac{1}{m_A} + \frac{1}{m_B}}$$

클램핑 (정지 마찰 한계):

$$j_t = \text{clamp}(j_t,\; -\mu\,j_n,\; \mu\,j_n)$$

여기서 $\mu$는 마찰 계수, $j_n$은 법선 임펄스 크기다.

```text
// 접선 마찰 적용 — invMass = 1/mass (§3 실전 코드와 동일한 스타일)
float relVt = (vx_A - vx_B) * tx + (vy_A - vy_B) * ty;
float jt = -relVt / invMassSum;
float maxFriction = mu * abs(j);  // j는 법선 임펄스
jt = clamp(jt, -maxFriction, maxFriction);

vx_A += jt * a.invMass * tx;
vy_A += jt * a.invMass * ty;
vx_B -= jt * b.invMass * tx;
vy_B -= jt * b.invMass * ty;   // 정적 물체(invMass=0)면 자동으로 미변경
```

**마찰 계수 예시:**

| 재질 | $\mu$ (정지) | $\mu$ (운동) |
|------|------------|-------------|
| 고무-콘크리트 | 1.0 | 0.8 |
| 금속-금속 | 0.6 | 0.4 |
| 얼음-얼음 | 0.1 | 0.03 |
| 캐릭터-바닥 (게임) | 0.3~0.5 | 0.2~0.4 |

---

## 5. 위치 보정 (Positional Correction)

임펄스만으로는 관통이 완전히 해소되지 않을 수 있다. 수치 적분의 불완전성 때문에 객체가 서로 파묻히는 현상이 발생하므로, **위치 보정**으로 강제로 밀어낸다.

$$\text{effectiveDepth} = \max(d - \text{slop},\, 0)$$

$$\Delta p_{\text{scalar}} = \frac{\text{effectiveDepth}}{1/m_A + 1/m_B} \cdot \text{factor} \quad (\text{factor} \approx 0.2 \sim 0.8)$$

$$\mathbf{p}_A' = \mathbf{p}_A + (\Delta p_{\text{scalar}} \cdot \text{invMass}_A)\,\mathbf{n}$$

$$\mathbf{p}_B' = \mathbf{p}_B - (\Delta p_{\text{scalar}} \cdot \text{invMass}_B)\,\mathbf{n}$$

```text
// 위치 보정 (Baumgarte) — invMass = 1/mass, 정적 물체는 invMass = 0 (§3 코드와 동일한 스타일)
float slop = 0.01f;            // 허용 오차
float correctionFactor = 0.8f; // 보정 강도
float correction = max(depth - slop, 0.0f)
                * correctionFactor / (a.invMass + b.invMass);

pA += correction * a.invMass * n;
pB -= correction * b.invMass * n;   // 정적 물체면 자동으로 0 → 미이동
```

> **슬랍(Slop)**: 아주 작은 관통($<$ slop)은 무시한다. 이는 물체가 바닥에 닿았을 때 미세하게 떠 있는 현상을 방지하고 안정성을 높인다(본 문서 §5 위치 보정의 `correction = max(depth - slop, 0)` 참조).

---

## 6. 안정성 팁

### Sleeping (휴면)

움직임이 거의 없는 물체는 물리 연산에서 제외한다. 속도와 각속도가 일정 임계값(예: 0.01) 이하로 오래 지속되면 sleep 상태로 전환하고, 큰 충격이 가해지면 다시 wake한다.

### 연속 충돌 검출 (CCD — Continuous Collision Detection)

빠르게 움직이는 물체(총알)가 얇은 벽을 통과하는 터널링 현상을 방지한다:

- **Sweep**: AABB/Sphere를 $dt$ 동안 밀어서 충돌 검사
- **Sub-step**: 프레임을 여러 sub-step으로 분할
- **Raycast**: 이동 궤적을 광선으로 검사

### 안정성 체크리스트

- [ ] $dt$ 상한 설정 (예: max 1/20초)
- [ ] sub-step 분할 (특히 스프링/관절)
- [ ] 반발 계수 $e \le 1$ 유지
- [ ] 마찰 임펄스가 법선 임펄스를 초과하지 않도록 clamp
- [ ] sleeping 상태에서도 약간의 damping 적용

---

## 7. 게임 활용

### 캐릭터 밀어내기 (Character Push)

플레이어가 NPC나 상자와 부딪혔을 때:

```text
// 플레이어 vs 상자 (e = 0, μ = 0.3)
resolveCollision(player.vel, massP, box.vel, massB,
                 contact.normal, 0.0f, 0.3f);
```

### 착지 (Landing)

캐릭터가 바닥에 닿았을 때 수직 속도를 처분:

```text
// 바닥 충돌 (e = 0, 완전 비탄성)
if (contact.normal.y > 0.7f) {  // 거의 수평면
    vel.y = max(vel.y, 0.0f);   // 수직 속도 제거
    isGrounded = true;
}
```

### 물리 장난감 — 포물선 공 던지기

공이 벽/바닥에 튕기며 반발:

```text
// 공 튕기기 (e = 0.5, μ = 0.2)
// 정적 벽은 §3처럼 wall.invMass = 0 으로 모델링
resolveCollision(ball.vel, ball.mass,
                 wall.vel, wall.invMass = 0,
                 wall.normal, 0.5f, 0.2f);
```

---

## 8. 빠른 참조

| 개념 | 수식 | 설명 |
|------|------|------|
| 임펄스 크기 | $j = \dfrac{-(1+e)\,v_n}{1/m_A + 1/m_B}$ | 충돌 임펄스 ($v_n$: 법선 상대 속도) |
| 속도 갱신 | $\mathbf{v}'_A = \mathbf{v}_A + \dfrac{j}{m_A}\mathbf{n}$ | 충돌 후 A의 속도 |
| 반발 계수 | $e = -\dfrac{v_n'}{v_n}$ | 충돌 후/전 법선 속도 비율 |
| 접선 임펄스 | $j_t = \text{clamp}(-\dfrac{\mathbf{v}_{rel}\cdot\mathbf{t}}{1/m_A+1/m_B},\; -\mu j_n,\; \mu j_n)$ | 마찰 임펄스 |
| Baumgarte | $\Delta\mathbf{p} = \mathbf{n}\cdot d \cdot \text{factor}$ | 위치 보정 |
| 접촉점 속도 | $\mathbf{v}_p = \mathbf{v}_{\text{COM}} + \boldsymbol{\omega} \times \mathbf{r}$ | 회전 포함 점 속도 |

| 용도 | $e$ | $\mu$ |
|------|-----|-------|
| 캐릭터-바닥 | 0 | 0.3~0.5 |
| 공 튕기기 | 0.3~0.8 | 0.1~0.3 |
| 금속 부딪힘 | 0.5~0.7 | 0.4~0.6 |
| 아이템 줍기 | 0 | 0.2 |
| 차량 충돌 | 0.1~0.3 | 0.6~0.8 |
