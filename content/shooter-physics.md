---
title: 슈팅 게임 물리 (Shooter Physics)
slug: shooter-physics
---

## 소개

슈팅 게임에서 총알이 어떻게 발사되고, 날아가고, 맞는지는 게임의 조작감과 재미를 결정짓는 핵심 요소다. 히트스캔(즉시 명중)부터 물리 기반 투사체까지, 다양한 탄도 방식과 반동/퍼짐/궤적 시각화를 다룬다.

> **핵심**
> 슈팅 물리는 **탄도 방식**(히트스캔 vs 투사체), **반동/퍼짐**(총기 조작감), **궤적 시각화**(유저 피드백) 세 축으로 구성된다.

> **참고**: 충돌 응답은 《충돌처리》, 광선 교차 검출은 《광선 교차》, 포물선 탄도는 《삼각함수 활용》, 난수 생성은 《난수 기초》를 각각 참고하라.

> **좌표계**: 본 문서는 **y-UP**(위가 $+y$)을 쓴다. 중력은 $g = -9.8\,\hat{y}$로 아래 방향. (《플랫포머 물리》는 y-DOWN 관례 — 문단 간 부호 차이에 유의.)

---

## 1. 탄도 방식 (Ballistic Methods)

### 히트스캔 (Hitscan)

발사 즉시 광선(Raycast)을 쏴서 명중 판정. 총알의 비행 시간이 없다.

```text
// 히트스캔 발사
vec3 origin = camera.position;
vec3 dir = camera.forward + getSpread();

RaycastHit hit;
if (raycast(origin, dir, maxRange, hit)) {
    // hit.point: 명중 위치
    // hit.normal: 충돌 법선
    // hit.entity: 명중한 객체
    applyDamage(hit.entity, damage);
    spawnImpactEffect(hit.point, hit.normal);
}
```

### 투사체 (Projectile / Physics-based)

총알에 질량과 속도를 부여하고 물리 엔진으로 매 프레임 갱신한다.

```text
// 투사체 발사
Bullet bullet;
bullet.position = muzzlePosition;
bullet.velocity = muzzleDirection * bulletSpeed;
bullet.gravity = vec3(0, -9.8, 0);

// 매 프레임
bullet.velocity += bullet.gravity * dt;
bullet.position += bullet.velocity * dt;
```

| 방식 | 장점 | 단점 |
|------|------|------|
| **히트스캔** | 즉각 반응, 구현 단순 | 총알 궤적 없음, 장거리 밸런스 |
| **투사체** | 현실적 궤적, 예측사 필요 | 지연 시간, 네트워크 동기화 어려움 |

> **하이브리드**: 겉으로는 투사체처럼 보이지만 실제 명중 판정은 히트스캔으로 하는 방식도 많다. 궤적은 클라이언트에서 시각용으로만 보여준다.

---

## 2. 총알 충돌 (Bullet Collision)

### 히트스캔 — Raycast

광선과 장면의 교차를 검사한다. 《광선 교차》 §1, §4 참고(예시 코드는 `intersectRayAABB`):

```text
// 광선 정의: O + t·D
// 모든 물체와의 교차 중 t > 0인 최소 t를 찾음
for (object in scene) {
    float t = intersectRayAABB(ray, object.bounds);
    if (t > 0 && t < bestT) {
        bestT = t;
        hitObject = object;
    }
}
```

### 투사체 — 연속 충돌 (Sweep / CCD)

투사체는 한 프레임에 여러 물체를 통과할 수 있다(터널링). 이를 방지하려면 **Sweep** 또는 **Sub-step**이 필요하다:

```text
// Sweep: 이전 위치 → 현재 위치를 AABB로 스윕
Bullet bullet;
vec3 prevPos = bullet.position;
bullet.position += bullet.velocity * dt;

// prevPos → bullet.position 구간을 스윕 충돌 검사
SweepHit hit;
if (sweepAABB(prevPos, bullet.position, bullet.halfSize, hit)) {
    // 충돌 발생
    bullet.position = hit.contactPoint;
    bullet.velocity = reflect(bullet.velocity, hit.normal);
}
```

> 상세 충돌 검출 알고리즘은 《충돌 검출》 §3(Sphere), §8(Capsule) 참고.

---

## 3. 반동 (Recoil)

총을 쏠 때 총구/조준점이 튀어오르는 현상. 대부분 스프링 또는 지수 감쇠로 복원한다.

### 스프링 기반 반동

《스프링 & 댐핑》 §3의 Semi-implicit Euler 적용:

```text
// 반동 추가 (일회성 충격)
recoilVelocity += vec2(
    randomRange(-0.5, 0.5) * recoilAmount,
    randomRange(0.5, 1.0) * recoilAmount
);

// 매 프레임 복원 (임계 또는 저댐핑)
void updateRecoil(float dt) {
    float k = 30.0f, c = 10.0f, m = 1.0f;
    vec2 force = -k * recoilOffset - c * recoilVelocity;
    recoilVelocity += (force / m) * dt;
    recoilOffset += recoilVelocity * dt;
}
```

### 지수 감쇠 기반 반동

가볍고 안정적인 대안. 《지수 감쇠》 참고:

```text
// 반동 즉시 적용
recoilOffset += vec2(randomRange(-0.5, 0.5), 1.0) * recoilAmount;

// 매 프레임 복원
float rate = 8.0f;
float t = 1.0f - exp(-rate * dt);
recoilOffset = lerp(recoilOffset, vec2(0, 0), t);
```

---

## 4. 퍼짐 (Spread)

총알이 조준점 정중앙이 아닌 일정 범위 내에서 무작위로 퍼져 나간다.

### 균일 분포 vs 가우시안 분포

```text
// 균일 분포 (원형)
float angle = randomRange(0, TWO_PI);
float radius = sqrt(randomRange(0, 1)) * maxSpread;
spreadOffset = vec2(cos(angle), sin(angle)) * radius;

// 가우시안 분포 (중앙 집중, 더 자연스러움)
// Box-Muller 변환 (《난수 기초》 참고)
float u1 = max(randomRange(0, 1), 1e-7f); // log(0) 방지
float u2 = randomRange(0, 1);
float gauss = sqrt(-2.0f * log(u1)) * cos(TWO_PI * u2);
spreadOffset = gauss * spreadFactor;
```

**퍼짐이 변하는 상황:**

```text
// 이동/사격 중 퍼짐 증가
float currentSpread = baseSpread;
currentSpread += velocity.len() * moveSpreadFactor;   // 이동 시
currentSpread += isFiring ? fireSpreadFactor : 0;      // 연사 시
currentSpread += isCrouching ? -crouchSpreadBonus : 0; // 앉기 시

// 매 프레임 서서히 복원 (지수 감쇠)
float t = 1.0f - exp(-recoveryRate * dt);
currentSpread = lerp(currentSpread, baseSpread, t);
```

---

## 5. 궤적 시각화 (Trajectory Visualization)

유저가 총알의 경로를 볼 수 있도록 궤적을 그린다.

### 포물선 궤적 (투사체 예측)

```text
// 미래 위치 예측 (베지어 또는 점진적)
void drawTrajectory(vec3 origin, vec3 velocity, float time) {
    for (float t = 0; t < maxTime; t += 0.05f) {
        vec3 pos = origin + velocity * t
                 + 0.5f * gravity * t * t;
        drawDot(pos, 0.1f);  // 점 그리기
    }
}
```

> 《삼각함수 활용》 탄도 절의 포물선 공식 참고. 곡선 렌더링은 《베지어 곡선》 참고.

### 레이저 조준선 (히트스캔)

```text
// 레이저 포인터 — 광선 끝까지 직선
vec3 end = origin + dir * maxRange;
drawLine(origin, end, color, 0.1f);
```

### 탄도 예측 — 타겟 리드 (Lead)

움직이는 적을 맞추기 위해 조준선 앞쪽을 예측. 총알이 발사 시점 $t$ 후에 적에게 닿는다면, 도달 조건은 "발사점과 적의 미래 위치 사이의 거리 = 총알 속도 × $t$":

$$|\mathbf{d} + \mathbf{v}_t \cdot t| = s \cdot t \quad (\mathbf{d} = \mathbf{pos}_{\text{target}} - \mathbf{pos}_{\text{origin}},\; s = \text{bulletSpeed})$$

양변을 제곱하면 $t$에 대한 2차 방정식이 된다 (시간 $t$에서의 위치 제곱 = 도달 거리 제곱):

$$(\mathbf{v}_t \cdot \mathbf{v}_t - s^2)\, t^2 + 2(\mathbf{v}_t \cdot \mathbf{d})\, t + (\mathbf{d} \cdot \mathbf{d}) = 0$$

```text
// 단순 리드: 적 속도 고려
float s = bulletSpeed;
vec3 d = target.position - origin;
float a = dot(target.velocity, target.velocity) - s * s;
float b = 2.0 * dot(target.velocity, d);
float c = dot(d, d);

float t = solveQuadratic(a, b, c);  // 양수 근 중 최소값 (없으면 리드 불가)
vec3 aimPoint = target.position + target.velocity * t;
```

---

## 6. 빠른 참조

| 개념 | 방식/수식 | 비고 |
|------|----------|------|
| 히트스캔 | 즉시 raycast | 지연 없음 |
| 투사체 | $\mathbf{p}' = \mathbf{p} + \mathbf{v}\,dt + \frac{1}{2}\mathbf{g}\,dt^2$ ※ | 중력 적용 |
| 반동 (스프링) | $F = -k x - c v$ | 《스프링 & 댐핑》 |
| 반동 (지수 감쇠) | $t = 1 - e^{-r\,dt}$ | 《지수 감쇠》 |
| 가우시안 퍼짐 | Box-Muller 변환 | 《난수 기초》 |
| 포물선 궤적 | $\mathbf{p}(t) = \mathbf{p}_0 + \mathbf{v}_0 t + 0.5\mathbf{g} t^2$ | 《삼각함수 활용》 |
| 리드 예측 | $\mathbf{aim} = \mathbf{pos}_{\text{target}} + \mathbf{v}_{\text{target}} \cdot t$ | 조준 보정 |

※ 해석 궤적식(닫힌 해). §1의 매 프레임 적분은 반암시적(semi-implicit) Euler($\mathbf{v} \mathrel{+}= \mathbf{g}\,dt$ 후 $\mathbf{p} \mathrel{+}= \mathbf{v}\,dt$)다.

| 총기 유형 | 반동 회복 | 퍼짐 | 탄도 |
|----------|----------|------|------|
| 권총 | 빠름 (rate 10~12) | 낮음 | 히트스캔 |
| 소총 | 중간 (rate 6~8) | 중간 | 히트스캔 |
| 저격총 | 느림 (rate 3~5) | 매우 낮음 | 히트스캔 |
| 산탄총 | 느림 | 높음 | 히트스캔 (다중) |
| 유탄/로켓 | 해당 없음 | 해당 없음 | 투사체 |
| 활/석궁 | 해당 없음 | 낮음 | 투사체 (중력) |
