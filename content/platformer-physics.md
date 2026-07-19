---
title: 플랫포머 물리 (Platformer Physics)
slug: platformer-physics
---

## 소개

플랫포머 게임의 캐릭터 물리는 "조작감"의 전부다. 가속/감속, 점프, 중력, 충돌, 그리고 이 모든 것이 결합된 **느낌**을 수학으로 정량화하고 튜닝하는 방법을 다룬다.

> **핵심**
> 플랫포머 물리의 핵심은 **가속 기반 이동**(정속이 아님), **가변 점프 높이**, **탄탄한 바닥 감지**다.

> **참고**: 충돌 응답은 《충돌처리》, AABB 충돌 검출은 《충돌 검출》, 지수 감쇠 카메라는 《지수 감쇠》를 각각 참고하라.

> **좌표계**: 본 문서는 **y-DOWN**(아래가 $+y$)을 쓴다. 중력은 $+y$ 방향, 점프 상승은 $-y$다. (《슈팅 게임 물리》《충돌처리》는 y-UP 관례 — 문단 간 부호 차이에 유의.)

---

## 1. 이동 모델 (Movement)

고전적인 플랫포머(마리오, 소닉 스타일)는 가속과 마찰로 속도를 제어한다.

### 가속/마찰/최대 속도

$$\mathbf{v} \mathrel{+}= \mathbf{accel} \cdot \text{dir} \cdot dt$$

$$\mathbf{v} = \text{clamp}(\mathbf{v}, -\mathbf{v}_{\max}, \mathbf{v}_{\max})$$

$$\mathbf{v} \mathrel{*}= \text{friction}^{\,dt \cdot 60}$$

**마찰을 프레임 독립적으로 적용**하는 이유: 단순히 `v *= friction`을 매 프레임 적용하면 fps에 따라 마찰력이 달라진다. 지수 감쇠와 동일한 원리다:

```text
// 프레임 독립 마찰
float f = pow(friction, dt * 60.0f);  // friction = 0.8~0.95
velocity.x *= f;

// 프레임 의존적 (잘못된 예)
velocity.x *= 0.9f;  // 60fps와 30fps에서 감속이 다름
```

```text
// 전체 이동 업데이트 (매 프레임)
void updateMovement(float dt) {
    // 가속
    float accel = 800.0f;    // 픽셀/초²
    float maxSpeed = 300.0f;
    velocity.x += accel * inputDir * dt;
    velocity.x = clamp(velocity.x, -maxSpeed, maxSpeed);

    // 마찰 (가속 입력이 없을 때만)
    if (inputDir == 0) {
        float friction = 0.85f;
        velocity.x *= pow(friction, dt * 60.0f);
    }

    // 위치 갱신
    position.x += velocity.x * dt;
}
```

| 파라미터 | 효과 | 일반적 범위 |
|---------|------|-----------|
| accel (가속) | 반응 속도 | 500~1500 |
| maxSpeed (최대 속도) | 최고 속력 | 200~500 |
| friction (마찰) | 정지할 때 미끄러짐 길이 | 0.8~0.95 (값이 1에 가까울수록 미끄러짐이 길다 — 속도 보존율, 1=무마찰) |

---

## 2. 점프 (Jumping)

### 기본 점프

중력과 초기 점프 속도로 점프 포물선을 만든다:

$$\text{gravity} = 2000\ \text{px/s}^2$$

$$\text{jumpVelocity} = -\sqrt{2 \cdot \text{gravity} \cdot \text{jumpHeight}}$$

```text
// 점프
if (isGrounded && jumpPressed) {
    velocity.y = -sqrt(2.0f * gravity * jumpHeight);
    isGrounded = false;
}

// 중력 적용 (매 프레임)
velocity.y += gravity * dt;
velocity.y = min(velocity.y, maxFallSpeed);  // 말단 속도 제한
position.y += velocity.y * dt;
```

### 말단 속도 (Max Fall Speed)

중력만 계속 적용하면 낙하 속도가 무한히 증가해 터널링과 비현실적 가속이 생긴다. **말단 속도**로 상한을 건다(y-DOWN 좌표계라 낙하 속도는 양수):

```text
velocity.y += gravity * dt;
velocity.y = min(velocity.y, maxFallSpeed);  // 예: 600~1000 px/s
```

### 가변 점프 높이 (Variable Jump Height)

버튼을 짧게 누르면 낮게, 길게 누르면 높게 점프. 핵심은 **상승 중 가속도 변경**:

```text
// 버튼을 누르고 있으면 추가 상승 가속
if (jumpHeld && velocity.y < 0) {
    velocity.y += jumpHoldAccel * dt;  // 음수 (위로)
}
// 버튼을 놓으면 중력 강화
else if (velocity.y < 0) {
    velocity.y += gravity * 2.0f * dt;  // 중력 2배
}
```

### 코요테 타임 (Coyote Time)

발이 땅에서 막 떨어진 직후(보통 50~100ms)에도 점프를 허용하는 관용 시간:

```text
coyoteTimer = isGrounded ? 0.1f : coyoteTimer - dt;
if (coyoteTimer > 0 && jumpPressed) {
    // 점프 허용
    isGrounded = false;
    coyoteTimer = 0;
}
```

### 점프 버퍼 (Jump Buffer)

플레이어가 아직 땅에 닿기 전에 점프 버튼을 미리 눌러도 점프가 실행되는 입력 예약:

```text
jumpBufferTimer = jumpPressed ? 0.1f : jumpBufferTimer - dt;
if (isGrounded && jumpBufferTimer > 0) {
    // 점프 실행
    jumpBufferTimer = 0;
}
```

---

## 3. 바닥/벽 충돌

### AABB 기반 땅 감지

캐릭터의 AABB 4꼭지점 아래로 약간(1~2px) 짧은 레이캐스트를 쏴서 바닥을 감지한다:

```text
// Ground Check
bool checkGrounded(vec2 pos, vec2 halfSize) {
    float checkDist = 2.0f;  // 약간의 여유
    vec2 origin = pos;
    origin.y += halfSize.y;  // AABB 하단

    // 좌/우 양측에서 체크
    for (float ox : {-halfSize.x + 1, halfSize.x - 1}) {
        origin.x = pos.x + ox;
        if (raycastDown(origin, checkDist)) {
            return true;
        }
    }
    return false;
}
```

> AABB 충돌의 자세한 내용은 《충돌 검출》 §2를 참고.

### 벽 슬라이딩 (Wall Sliding)

벽에 닿았을 때 수직 이동은 유지하면서 수평만 막는다. 《거리 & 최단점》 §9 참고:

```text
// 수평 충돌 후 속도 처리
if (collisionLeft || collisionRight) {
    velocity.x = 0;
    // 수직 속도는 유지 (점프 중 벽에 붙어서 미끄러짐)
}
```

---

## 4. 경사/계단 처리

### 한계 각도 (Slope Limit)

캐릭터가 오를 수 있는 최대 경사각. 보통 45°~60°:

```text
// 경사면 법선으로부터 각도 계산
float slopeAngle = acos(dot(slopeNormal, vec2(0, -1)));
float maxSlope = radians(45.0f);

if (slopeAngle > maxSlope) {
    // 너무 가파름 → 벽처럼 처리
    velocity.x = 0;
} else {
    // 경사면 따라 이동
    velocity = projectOnSlope(velocity, slopeNormal);
}
```

### 스텝 업 (Step Up)

아주 작은 턱(보통 캐릭터 키의 1/4 이하)은 자동으로 올라간다:

```text
// 수평 이동 후 충돌 발생 시, 작은 턱인지 확인
if (horizontalCollision) {
    // 현재 위치 + stepHeight만큼 위에서 다시 체크
    vec2 raisedPos = position;
    raisedPos.y -= stepHeight;
    if (!checkCollision(raisedPos)) {
        position.y -= stepHeight;  // 턱 올라가기
    }
}
```

---

## 5. 카메라 따라가기 (Camera Follow)

지수 감쇠를 사용해 카메라가 플레이어를 부드럽게 따라간다. 자세한 내용은 《지수 감쇠》 참고:

```text
// 지수 감쇠 카메라
float rate = 4.0f;  // 클수록 빠름
float t = 1.0f - exp(-rate * dt);
camera.pos = lerp(camera.pos, targetPos, t);
```

**데드존(Dead Zone)**: 플레이어가 화면 중앙의 작은 영역 안에 있으면 카메라는 움직이지 않는다:

```text
vec2 delta = targetPos - camera.pos;
if (length(delta) > deadZoneRadius) {
    // 데드존 밖으로 나가면 카메라 따라감
    camera.pos += delta * t;  // 또는 lerp
}
```

---

## 6. 빠른 참조

| 요소 | 수식/방법 | 비고 |
|------|----------|------|
| 가속 이동 | $\mathbf{v} \mathrel{+}= \text{accel} \cdot \text{dir} \cdot dt$ | 프레임 독립 |
| 마찰 | $\mathbf{v} \mathrel{*}= \text{pow}(\text{friction},\ dt \cdot 60)$ | 프레임 독립 |
| 점프 속도 | $\text{jumpVel} = -\sqrt{2 \cdot g \cdot h}$ | $h$: 목표 높이 |
| 말단 속도 | $\text{velocity.y} = \min(\text{velocity.y},\ \text{maxFallSpeed})$ | 낙하 속도 상한 |
| 가변 점프 | 버튼 홀드 시 추가 가속 | 중력 2배로 하강 |
| 코요테 타임 | 공중 점프 관용 (낙하 직후) | 50~100ms |
| 점프 버퍼 | 착지 전 입력 예약 | 50~100ms |
| 바닥 감지 | AABB 하단 일부 raycast | 여유 1~2px |
| 경사 한계 | $\text{acos}(\mathbf{n} \cdot \text{up})$ | 45°~60° |
| 스텝 업 | 충돌 시 +stepHeight 재검사 | 키의 1/4 |
| 카메라 | $t = 1 - e^{-r\,dt}$ | 지수 감쇠 |

| 파라미터 | 일반값 | 설명 |
|---------|-------|------|
| gravity | 1500~2500 | 중력 가속도 (px/s²) |
| jumpHeight | 80~150 | 최대 점프 높이 (px) |
| maxSpeed | 200~500 | 최대 이동 속도 (px/s) |
| maxFallSpeed | 600~1000 | 말단 속도 (px/s) |
| accel | 500~1500 | 가속도 (px/s²) |
| friction | 0.8~0.95 | 마찰 계수 |
| coyoteTime | 0.05~0.1 | 코요테 타임 (초) |
| jumpBuffer | 0.05~0.1 | 점프 버퍼 (초) |
| stepHeight | 16~24 | 자동 스텝업 높이 (px) |
