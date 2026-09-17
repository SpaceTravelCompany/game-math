---
title: 지수 감쇠 (Exponential Smoothing)
slug: exponential-smoothing
---

## 소개

`lerp(current, target, 0.1)`처럼 고정 비율로 보간하면 프레임률(FPS)에 따라 이동 속도가 달라진다. 144Hz 모니터에서는 너무 빠르고, 30fps로 떨어지면 굼떠진다.

**지수 감쇠(Exponential Smoothing)**는 프레임 시간(`dt`)을 고려하여 **어떤 FPS에서도 항상 일정하고 부드럽게 목표에 도달**하게 만드는 필수 기법이다.

> **핵심 요약**  
> `lerp`의 세 번째 인자로 고정값 대신 **`1.0 - exp(-decay * dt)`** 를 넣으면 끝난다.

---

## 1. 왜 고정 Lerp는 문제가 되는가?

매 프레임 10%씩 다가가는 코드를 생각해보자:

```text
// ❌ 프레임 의존적: FPS마다 속도가 달라짐
position = lerp(position, target, 0.1);
```

- **60 FPS 환경 (1초에 60번 실행)**: 남은 거리는 $0.9^{60} \approx 0.18\%$ (목표에 거의 다 감)
- **30 FPS 환경 (1초에 30번 실행)**: 남은 거리는 $0.9^{30} \approx 4.2\%$ (목표에 덜 감)

똑같이 1초가 흘렀는데도 프레임이 낮으면 더 느리게 움직인다. 이것이 게임에서 흔히 일어나는 조작감 버그의 원인이다.

---

## 2. 해결책: 프레임 독립 공식

```text
//  프레임 독립적: FPS와 무관하게 일정한 속도
float t = 1.0 - exp(-decayRate * dt);
position = lerp(position, target, t);
```

### 왜 이 공식이 작동할까? (직관)
시간이 흐르면 남은 거리는 지수적으로 줄어든다 ($e^{-\text{decay} \cdot dt}$).  
따라서 "목표를 향해 이동한 비율"은 전체 $1$에서 남은 비율을 뺀 **$1 - e^{-\text{decay} \cdot dt}$** 가 된다.

- $dt$가 작으면 (고주사율): 잘게 쪼개서 조금씩 이동
- $dt$가 크면 (렉/저사양): 한 번에 그만큼 큼직하게 이동
- **결과**: 1초 동안 실제 움직인 거리는 30 FPS든 144 FPS든 완전히 똑같다.

---

## 3. `decayRate` 감각 익히기

`decayRate`가 클수록 목표에 빠르게 달라붙는다.

| `decayRate` | 체감 속도 | 반감기 (거리 절반에 걸리는 시간) | 추천 용도 |
|:---:|:---:|:---:|:---|
| **$2 \sim 4$** | 느긋하고 묵직함 | 약 $0.17 \sim 0.35$초 | HP 게이지 감소, 조명 전환 |
| **$5 \sim 8$** | 부드럽고 자연스러움 | 약 $0.09 \sim 0.14$초 | 3인칭 카메라 추적, 미니맵 |
| **$10 \sim 15$** | 빠르고 경쾌함 | 약 $0.05 \sim 0.07$초 | UI 팝업, 무기 에임 복원 |
| **$20+$** | 거의 즉각 반응 | $0.03$초 이하 | 입력 지연 없는 빠른 조준선 |

> **반감기(Half-Life) 팁**  
> 남은 거리가 딱 절반으로 줄어드는 데 걸리는 시간은 $t_{1/2} = \dfrac{\ln 2}{\text{decayRate}} \approx \dfrac{0.693}{\text{decayRate}}$ 이다.  
> 예를 들어 decay가 $7$이면 약 $0.1$초마다 거리가 반씩 줄어든다.

---

## 4. 실무 코드 패턴

### 카메라 따라가기 (Smooth Follow)

카메라가 플레이어를 부드럽게 뒤따라갈 때 가장 흔하게 쓰인다:

```text
void updateCamera(float dt) {
    float decay = 6.0f; // 부드러운 카메라 추종
    float t = 1.0f - exp(-decay * dt);
    cameraPos = lerp(cameraPos, playerPos, t);
}
```

### 쿼터니언 회전 감쇠

방향이나 회전을 부드럽게 돌릴 때는 `slerp`와 결합한다:

```text
float t = 1.0f - exp(-8.0f * dt);
currentRotation = slerp(currentRotation, targetRotation, t);
```

### 체력바(HP Bar) 지연 효과

피격 시 즉시 깎이는 빨간 바 뒤로, 서서히 따라오는 흰색 잔상 바:

```text
// actualHP: 실제 현재 체력, displayHP: 화면에 보이는 체력바
float t = 1.0f - exp(-4.0f * dt);
displayHP = lerp(displayHP, actualHP, t);
```

---

## 5. 지수 감쇠 vs 스프링(Spring) vs SmoothDamp

언제 무엇을 골라야 할까?

```text
지수 감쇠:    ─── 목표를 향해 미끄러지듯 스르륵 멈춤 (오버슈트 없음, 가장 단순)
스프링:      ─── 퉁~ 하고 튕겼다가 제자리로 복원 (탄성, 젤리 느낌)
SmoothDamp:  ─── 부드럽게 출발했다가 감속하며 정지 (출발도 부드러움)
```

| 기법 | 파라미터 | 관성/튕김 | 적합한 곳 |
|---|:---:|:---:|:---|
| **지수 감쇠** | `decayRate` 1개 | ❌ 없음 | 카메라 추적, 체력바, 회전 감쇠, 범용 UI |
| **스프링** | 강성($k$), 감쇠($c$) | ⭕ 있음 | 총기 반동 복원, 흔들리는 젤리, 밧줄 |
| **SmoothDamp** | 도달시간(`smoothTime`) | ❌ (임계 감쇠) | 시네마틱 카메라, 고급 UI 트랜지션 |

---

## 6. 빠른 참조

```text
// 범용 지수 감쇠 보간
function damp(source, target, decay, dt):
    return lerp(source, target, 1.0 - exp(-decay * dt))
```

| 상황 | 공식 / 코드 |
|---|---|
| 기본 보간 계수 | `float t = 1.0 - exp(-decay * dt)` |
| 1D 수치 보간 | `val = lerp(val, target, 1.0 - exp(-decay * dt))` |
| 3D 벡터 보간 | `pos = lerp(pos, targetPos, 1.0 - exp(-decay * dt))` |
| 회전 보간 | `rot = slerp(rot, targetRot, 1.0 - exp(-decay * dt))` |
| 반감기 공식 | $t_{1/2} = 0.693 / \text{decay}$ |
