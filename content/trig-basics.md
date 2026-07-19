---
title: 삼각함수 기초 (Trigonometry Basics)
slug: trig-basics
---

## 소개

삼각함수는 각도와 길이의 관계를 다루는 수학이다. 게임에서 회전, 파동, 원운동, 탄도 계산 등 거의 모든 곳에서 사용된다.

> **핵심**
> sin은 높이(Y), cos은 밑변(X), tan은 기울기다. 단위원에서 모든 것이 시작된다.

---

## 1. 각도와 라디안

### 정의

$$1 \text{ 라디안} = \text{호의 길이가 반지름과 같은 각도}$$

$$\text{원 } 1 \text{ 바퀴} = 2\pi \text{ rad} = 360°$$

### 변환

$$\text{라디안} = \text{도} \times \frac{\pi}{180}$$

$$\text{도} = \text{라디안} \times \frac{180}{\pi}$$

> **자주 쓰는 값**

| 도 | 라디안 |
|---|---|
| $0°$ | $0$ |
| $30°$ | $\pi/6$ |
| $45°$ | $\pi/4$ |
| $60°$ | $\pi/3$ |
| $90°$ | $\pi/2$ |
| $180°$ | $\pi$ |
| $270°$ | $3\pi/2$ |
| $360°$ | $2\pi$ |

> **실전 팁**: 게임 내부 계산은 항상 라디안을 사용한다. UI 표시용으로만 도(degree)로 변환한다. 대부분의 수학 함수(sin, cos, atan2 등)는 라디안 입력을 받는다.

> **왜 라디안인가**: 호의 길이 $s = r\theta$는 $\theta$가 라디안일 때만 그대로 성립한다(도 단위는 $\pi/180$ 보정 필요). 미적분과 테일러 전개($\sin x \approx x - x^3/6$), 각속도(rad/s)도 라디안에서 자연스럽다.

---

## 2. 단위원 (Unit Circle)

반지름이 1인 원에서 각도에 따른 좌표를 정의한다.

> 각도 $\theta$에서 단위원 위의 점

$$x = \cos(\theta)$$

$$y = \sin(\theta)$$

| 각도 | 좌표 | cos | sin |
|------|------|-----|-----|
| $0°$ | $(1, 0)$ | $\cos(0) = 1$ | $\sin(0) = 0$ |
| $90°$ | $(0, 1)$ | $\cos(90°) = 0$ | $\sin(90°) = 1$ |
| $180°$ | $(-1, 0)$ | $\cos(180°) = -1$ | $\sin(180°) = 0$ |
| $270°$ | $(0, -1)$ | $\cos(270°) = 0$ | $\sin(270°) = -1$ |

```text
         90° (0, 1)
          |
          |
180° ------+------ 0° (1, 0)
(-1, 0)   |
          |
         270° (0, -1)
```

### 임의 반지름 r인 경우

$$x = r \times \cos(\theta)$$

$$y = r \times \sin(\theta)$$

### 사분면 부호

각 사분면에서 $(\cos, \sin, \tan)$의 부호:

| 사분면 | 각도 범위 | $(\cos, \sin, \tan)$ |
|--------|-----------|----------------------|
| Q1 | $0 \sim \pi/2$ | $(+, +, +)$ |
| Q2 | $\pi/2 \sim \pi$ | $(-, +, -)$ |
| Q3 | $\pi \sim 3\pi/2$ | $(-, -, +)$ |
| Q4 | $3\pi/2 \sim 2\pi$ | $(+, -, -)$ |

> **패턴**: sin은 위 반원(Q1·Q2)에서 $+$, cos은 오른쪽 반원(Q1·Q4)에서 $+$. tan은 cos·sin의 부호가 같은 Q1·Q3에서 $+$, 다른 Q2·Q4에서 $-$다.

---

## 3. 사인, 코사인, 탄젠트

### 직각삼각형 정의

> 각도 $\theta$에 대해

$$\sin(\theta) = \frac{\text{높이}}{\text{빗변}} = \frac{\text{opposite}}{\text{hypotenuse}}$$

$$\cos(\theta) = \frac{\text{밑변}}{\text{빗변}} = \frac{\text{adjacent}}{\text{hypotenuse}}$$

$$\tan(\theta) = \frac{\text{높이}}{\text{밑변}} = \frac{\text{opposite}}{\text{adjacent}} = \frac{\sin(\theta)}{\cos(\theta)}$$

### 역수 관계

$$\csc(\theta) = \frac{1}{\sin(\theta)} \quad \text{(코시컨트)}$$

$$\sec(\theta) = \frac{1}{\cos(\theta)} \quad \text{(시컨트)}$$

$$\cot(\theta) = \frac{1}{\tan(\theta)} \quad \text{(코탄젠트)}$$

---

## 4. 핵심 공식

### 피타고라스 항등식

$$\sin^2(\theta) + \cos^2(\theta) = 1$$

### 덧셈 공식

$$\sin(a + b) = \sin(a)\cos(b) + \cos(a)\sin(b)$$

$$\cos(a + b) = \cos(a)\cos(b) - \sin(a)\sin(b)$$

$$\tan(a + b) = \frac{\tan(a) + \tan(b)}{1 - \tan(a)\tan(b)}$$

### 배각 공식

$$\sin(2\theta) = 2\sin(\theta)\cos(\theta)$$

$$\cos(2\theta) = \cos^2(\theta) - \sin^2(\theta) = 2\cos^2(\theta) - 1 = 1 - 2\sin^2(\theta)$$

### 반각 공식

$$\sin\left(\frac{\theta}{2}\right) = \pm\sqrt{\frac{1 - \cos(\theta)}{2}}$$

$$\cos\left(\frac{\theta}{2}\right) = \pm\sqrt{\frac{1 + \cos(\theta)}{2}}$$

---

## 5. 역삼각함수

각도를 알고 싶을 때 사용한다.

### 기본 역삼각함수

$$\arcsin(x) = \sin^{-1}(x) \quad x \in [-1, 1] \to [-\pi/2, \pi/2]$$

$$\arccos(x) = \cos^{-1}(x) \quad x \in [-1, 1] \to [0, \pi]$$

$$\arctan(x) = \tan^{-1}(x) \quad x \in \mathbb{R} \to (-\pi/2, \pi/2)$$

### atan2 — 게임에서 가장 중요한 역삼각함수

$$\text{atan2}(y, x) \to [-\pi, \pi]$$

> 일반 atan과의 차이:

$$\text{atan}(y/x) \to (-\pi/2, \pi/2) \quad \text{(사분면 구분 불가)}$$

$$\text{atan2}(y, x) \to [-\pi, \pi] \quad \text{(정확한 사분면 반환)}$$

```text
// atan2 사분면
        y
        |
   Q2   |   Q1
 --------+-------- x
   Q3   |   Q4
        |
```

> **예시**

$$\text{atan2}(1, 1) = 45° \quad \text{(Q1)}$$

$$\text{atan2}(1, -1) = 135° \quad \text{(Q2)}$$

$$\text{atan2}(-1, -1) = -135° \quad \text{(Q3)}$$

$$\text{atan2}(-1, 1) = -45° \quad \text{(Q4)}$$

> **실전 팁**: 각도를 구할 때 항상 atan2를 사용하자. atan(y/x)는 비율 $y/x$만 보존하므로 서로 반대편 사분면(예 Q1↔Q3)을 구별하지 못한다.

> **주의 — 인수 순서 이식성**: 본 문서와 GLSL `atan(y, x)`는 $y$를 먼저 받는다(표준 C `atan2(y, x)`와 동일). 그러나 **Excel/LibreOffice `ATAN2(x_num, y_num)`은 $x$를 먼저 받는다(반전)**. 일부 언어 래퍼도 순서가 다를 수 있으므로, 이식할 때 반드시 시그니처를 확인할 것.

---

## 6. 사인/코사인 값 표

| 각도 | 라디안 | sin | cos | tan |
|------|--------|-----|-----|-----|
| 0° | 0 | 0 | 1 | 0 |
| 30° | π/6 | 0.5 | √3/2 ≈ 0.866 | 1/√3 ≈ 0.577 |
| 45° | π/4 | √2/2 ≈ 0.707 | √2/2 ≈ 0.707 | 1 |
| 60° | π/3 | √3/2 ≈ 0.866 | 0.5 | √3 ≈ 1.732 |
| 90° | π/2 | 1 | 0 | ∞ |
| 180° | π | 0 | -1 | 0 |
| 270° | 3π/2 | -1 | 0 | ∞ |

---

## 7. 삼각함수의 주기성

$$\sin(\theta + 2\pi) = \sin(\theta) \quad \text{(주기 } 2\pi\text{)}$$

$$\cos(\theta + 2\pi) = \cos(\theta) \quad \text{(주기 } 2\pi\text{)}$$

> **대칭**

$$\sin(-\theta) = -\sin(\theta) \quad \text{(기함수, 원점 대칭)}$$

$$\cos(-\theta) = \cos(\theta) \quad \text{(우함수, y축 대칭)}$$

> **보각**

$$\sin(\pi - \theta) = \sin(\theta)$$

$$\cos(\pi - \theta) = -\cos(\theta)$$

---

## 8. 게임에서의 기본 활용

### 회전

> 2D 회전: 각도 → 방향 벡터

$$\text{dirX} = \cos(\text{angle})$$

$$\text{dirY} = \sin(\text{angle})$$

> 이동

$$\text{velocity}_x = \text{speed} \times \cos(\text{angle})$$

$$\text{velocity}_y = \text{speed} \times \sin(\text{angle})$$

### 거리와 각도

> 두 점 사이의 각도

$$\text{angle} = \text{atan2}(\text{target}_y - \text{source}_y,\ \text{target}_x - \text{source}_x)$$

> 거리

$$\text{distance} = \sqrt{dx^2 + dy^2}$$

### 삼각형 면적

> 두 변과 끼인각으로 면적

$$\text{area} = 0.5 \times a \times b \times \sin(\theta)$$

---

## 9. 빠른 참조

| 함수 | 정의 | 범위 | 활용 |
|------|------|------|------|
| sin | 높이/빗변 | [-1, 1] | Y축, 파동 |
| cos | 밑변/빗변 | [-1, 1] | X축, 진동 |
| tan | 높이/밑변 | (-∞, ∞) | 기울기 |
| asin | sin의 역 | [-π/2, π/2] | 각도 복원 |
| acos | cos의 역 | [0, π] | 내적→각도 |
| atan2 | atan2(y, x): 사분면 정확한 각도 | [-π, π] | 방향 각도 |

| 항등식 | 공식 |
|--------|------|
| 피타고라스 | sin²+cos²=1 |
| 배각 | sin(2θ)=2sinθcosθ |
| 사분면 | atan2로 정확한 각도 |
| 주기 | sin(θ+2π)=sin(θ) |