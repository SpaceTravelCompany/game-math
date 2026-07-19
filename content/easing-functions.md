---
title: Easing 함수 (Easing Functions)
slug: easing-functions
---

## 소개

Easing 함수는 애니메이션의 시작과 끝을 자연스럽게 만드는 수학 함수다. 선형 이동은 기계적으로 보이지만, Easing을 적용하면 물리적으로 자연스러운 움직임이 된다.

> **핵심**
> 선형은 기계적, Easing은 자연스럽다. 가속/감속이 부드러운 느낌을 만든다.

---

## 1. 선형 (Linear)

일정한 속도. 가장 단순하지만 기계적으로 보인다.

$$\text{linear}(t) = t$$

**게임에서의 활용:**
- 기계적인 이동 (컨베이어 벨트)
- 회전하는 톱
- 로딩 바 (균일한 진행)

> **주의**: 캐릭터나 카메라에 선형을 쓰면 부자연스럽다. 특히 시작/끝이 딱 끊어지는 느낌이 든다.

---

## 2. 2차 (Quadratic) Easing

### Ease-In Quad (점점 빠름)

$$\text{easeInQuad}(t) = t^2$$

천천히 시작해서 점점 빨라진다. 자유낙하, 중력 효과.

### Ease-Out Quad (점점 느림)

$$\text{easeOutQuad}(t) = 1 - (1 - t)^2$$

빨리 시작해서 천천히 멈춘다. 마찰, 감속.

### Ease-In-Out Quad (가속 → 감속)

$$\text{easeInOutQuad}(t) = \begin{cases} 2t^2 & \text{if } t < 0.5 \\ 1 - 2(1-t)^2 & \text{otherwise} \end{cases}$$

처음엔 가속, 나중엔 감속. 자연스러운 이동.

---

## 3. 3차 (Cubic) Easing

2차보다 더 강한 가속/감속.

$$\text{easeInCubic}(t) = t^3$$

$$\text{easeOutCubic}(t) = 1 - (1-t)^3$$

$$\text{easeInOutCubic}(t) = \begin{cases} 4t^3 & \text{if } t < 0.5 \\ 1 - 4(1-t)^3 & \text{otherwise} \end{cases}$$

### 3차 vs 2차

3차가 더 "드라마틱" (느린 시작, 빠른 중간, 느린 끝), 2차는 더 미묘한 변화. 게임에서 보통 3차를 기본으로 사용한다.

---

## 4. 지수 (Exponential) Easing

훨씬 더 극적인 가속/감속.

$$\text{easeInExpo}(t) = 2^{10(t-1)}$$

$$\text{easeOutExpo}(t) = 1 - 2^{-10t}$$

**게임에서의 활용:**
- **순간이동 효과**: 급출발 후 멈춤
- **화면 전환**: 메뉴가 확 나타남
- **파티클**: 폭발 초기 확장

---

## 5. Back Easing (뒤로 당겨서 쏘기)

시작/끝에서 약간 뒤로 넘어갔다가 진행된다.

### Ease-In Back

$$\text{easeInBack}(t) = 2.70158 \times t^3 - 1.70158 \times t^2$$

$c_1 = 1.70158$ (기본 상수, 더 크게 하면 더 많이 뒤로 넘어감)

### Ease-Out Back

$$c_1 = 1.70158, \quad c_3 = c_1 + 1$$

$$\text{easeOutBack}(t) = 1 + c_3 \times (t-1)^3 + c_1 \times (t-1)^2$$

### Ease-In-Out Back

$$c_1 = 1.70158, \quad c_2 = c_1 \times 1.525$$

$$\text{easeInOutBack}(t) = \begin{cases} \dfrac{(2t)^2\bigl((c_2+1)\cdot 2t - c_2\bigr)}{2} & \text{if } t < 0.5 \\[6pt] \dfrac{(2t-2)^2\bigl((c_2+1)(2t-2) + c_2\bigr) + 2}{2} & \text{otherwise} \end{cases}$$

**게임에서의 활용:**
- **버튼 클릭**: 눌렀다가 튕겨나옴
- **메뉴 등장**: 뒤에서 살짝 당겨졌다가 튀어나옴
- **점수 팝업**: 숫자가 튕기면서 나타남
- **캐릭터 대쉬**: 뒤로 살짝 빠졌다가 질주

---

## 6. Elastic Easing (탄성)

고무줄처럼 진동하며 목표에 도달한다.

### Ease-Out Elastic

$$c_4 = \frac{2\pi}{3}$$

$$\text{easeOutElastic}(t) = \begin{cases} 0 & \text{if } t = 0 \\ 1 & \text{if } t = 1 \\ 2^{-10t} \times \sin\!\left((10t - 0.75) \times c_4\right) + 1 & \text{otherwise} \end{cases}$$

### Ease-In Elastic

$$c_4 = \frac{2\pi}{3}$$

$$\text{easeInElastic}(t) = \begin{cases} 0 & \text{if } t = 0 \\ 1 & \text{if } t = 1 \\ -2^{10t-10} \times \sin\!\left((10t - 10.75) \times c_4\right) & \text{otherwise} \end{cases}$$

**게임에서의 활용:**
- **게임오버**: 화면이 흔들리며 멈춤
- **콤보 표시**: 점수가 튕기며 올라감
- **아이템 획득**: 효과음과 함께 팝업이 진동
- **UI 등장**: 요소가 탄력 있게 나타남

---

## 7. Bounce Easing (바운스)

공이 튕기듯 여러 번 바운스하며 멈춘다.

### Ease-Out Bounce

```text
function easeOutBounce(t):
    n1 = 7.5625
    d1 = 2.75

    if t < 1/d1:
        return n1 × t × t
    elif t < 2/d1:
        t -= 1.5/d1
        return n1 × t × t + 0.75      // 반환 상수 0.75 = 3/4
    elif t < 2.5/d1:
        t -= 2.25/d1
        return n1 × t × t + 0.9375    // 반환 상수 0.9375 = 15/16
    else:
        t -= 2.625/d1
        return n1 × t × t + 0.984375  // 반환 상수 0.984375 = 63/64

    // 각 바운스의 감쇠율: ~50%
```

### Ease-In Bounce

$$\text{easeInBounce}(t) = 1 - \text{easeOutBounce}(1 - t)$$

**게임에서의 활용:**
- **볼/공**: 튕기며 멈추는 효과
- **코인 드롭**: 아이템이 바닥에 떨어져 튕김
- **캐릭터 점프**: 착지 시 튕김 효과
- **게임오버 텍스트**: 튕기며 나타남

---

## 8. Easing 선택 가이드

| 상황 | 추천 Easing | 이유 |
|------|------------|------|
| 자연스러운 이동 | Ease-In-Out Cubic | 가속→감속 |
| 멈추는 이동 | Ease-Out Quad/Cubic | 빠르게 가다 감속 |
| 출발하는 이동 | Ease-In Quad/Cubic | 천천히 가다 가속 |
| UI 팝업 | Ease-Out Back | 뒤로 당겨졌다가 등장 |
| 버튼 클릭 | Ease-In-Out Back | 자연스러운 눌림 |
| 콤보/점수 | Ease-Out Elastic | 튕기는 효과 |
| 공 튕김 | Ease-Out Bounce | 물리적 바운스 |
| 폭발/충격 | Ease-Out Expo | 급출발 후 감속 |
| 화면 전환 | Ease-In-Out Quad | 부드러운 전환 |
| 기계적 이동 | Linear | 의도된 기계적 느낌 |

---

## 9. 커스텀 Easing

### Bezier 기반 Easing

CSS의 cubic-bezier와 같은 방식: $\text{cubic-bezier}(x_1, y_1, x_2, y_2)$. 예: $\text{cubic-bezier}(0.25,\, 0.1,\, 0.25,\, 1)$ — "ease" 키워드. $t$를 곡선을 통해 매핑하며, $x$축은 시간($0 \to 1$), $y$축은 진행도($0 \to 1$)이다.

### 게임에서의 커스텀 Easing

```text
// 진폭과 주기를 조절한 탄성
function customElastic(t, amplitude, period):
    if t == 0 or t == 1: return t
    s = period / 4    // amplitude = 1 전용 공식
    // amplitude ≠ 1이면: s = period × arcsin(1/amplitude) / (2π)
    return amplitude × 2^(-10t) × sin((t - s) × 2π / period) + 1
```

---

## 10. 실전 예시: 카메라 줌

```text
// 스나이퍼 조준 시 카메라 줌
function updateZoom(dt):
    targetFov = isAiming ? 30 : 70

    // Ease-Out Cubic으로 부드럽게
    t = 1 - exp(-zoomSpeed × dt)    // 프레임 독립적
    currentFov = lerp(currentFov, targetFov, easeOutCubic(t))

    camera.fov = currentFov
```

---

## 11. 실전 예시: 점수 팝업

```text
function showScorePopup(value, position):
    popup = createPopup(value, position)
    popup.age = 0
    popup.duration = 1.2

function updatePopup(popup, dt):
    popup.age += dt
    t = clamp(popup.age / popup.duration, 0, 1)

    // 등장: Ease-Out Back (0~0.3)
    // 유지: 1 (0.3~0.9)
    // 퇴장: Ease-In Quad (0.9~1.0)

    if t < 0.3:
        scale = easeOutBack(t / 0.3)
    elif t < 0.9:
        scale = 1
        popup.y += dt × 30    // 천천히 위로
    else:
        scale = 1
        opacity = 1 - easeInQuad((t - 0.9) / 0.1)
```

---

## 12. 빠른 참조

| Easing | 공식 (간략) | 느낌 |
|--------|------------|------|
| Linear | $t$ | 기계적 |
| Quad | $t^2$ | 부드러운 가속/감속 |
| Cubic | $t^3$ | 더 강한 가속/감속 |
| Expo | $2^{10t-10}$ | 극적 |
| Back | $t^3 + \text{overshoot}$ | 당겨서 쏘기 |
| Elastic | $\exp \times \sin$ | 고무줄 |
| Bounce | 다단계 2차 | 공 튕김 |
| Smoothstep | $t^2(3-2t)$ | 범용 부드러움 |

> Smoothstep의 정의와 게임 활용은 **《삼각함수 활용》 문서의 Smoothstep 절(§6)**에 있으므로 참고.