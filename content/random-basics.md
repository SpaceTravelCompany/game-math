---
title: 난수 기초 (Random Number Basics)
slug: random-basics
---

## 소개

난수는 게임에서 재미, 다양성, 예측 불가능성을 만드는 핵심 요소다. 아이템 드롭, 맵 생성, AI 행동, 데미지 변동 등 거의 모든 시스템에 사용된다. 하지만 잘못된 난수 사용은 치명적인 버그를 만들 수 있다.

> **핵심**
> 시드(Seed)만 같으면 같은 난수 시퀀스가 재현된다. 게임에서는 결정론적(deterministic) 난수가 중요하다.

---

## 1. 균등 난수 (Uniform Random)

가장 기본적인 난수로, 주어진 범위 내에서 모든 값이 같은 확률로 나온다.

### 0~1 실수 난수

$$r = \text{random()} \quad r \in [0, 1)$$

### 범위 지정

$$r = \text{min} + \text{random()} \times (\text{max} - \text{min}) \quad r \in [\text{min}, \text{max})$$

$$i = \text{min} + \lfloor \text{random()} \times (\text{max} - \text{min} + 1) \rfloor \quad i \in [\text{min}, \text{max}]$$

### 정수 범위 (모듈러)

$$i = \text{randomInt()} \bmod n \quad i \in [0, n)$$

> 주의: 모듈러 방식은 약간의 편향이 있을 수 있음 (난수 생성기의 최대값이 n의 배수가 아닌 경우)

---

## 2. 시드 (Seed)와 결정론

### 시드의 역할

```text
// 같은 시드 → 같은 난수 시퀀스
setSeed(42)
r1 = random()    // 0.7234...
r2 = random()    // 0.1893...

setSeed(42)      // 다시 같은 시드
r3 = random()    // 0.7234... (r1과 동일!)
r4 = random()    // 0.1893... (r2와 동일!)
```

### 게임에서 시드가 중요한 이유

```text
// 1. 멀티플레이어 동기화
// 모든 플레이어가 같은 시드 → 같은 맵, 같은 드롭
seed = 12345
generateMap(seed)

// 2. 리플레이 / 데모
// 시드만 저장하면 같은 게임 재현 가능

// 3. 프로시저럴 생성
// "월드 시드"로 같은 월드 재생성
worldSeed = 78901
terrain = generateTerrain(worldSeed)
```

> **실전 팁**: 멀티플레이어 게임에서는 반드시 각 클라이언트가 같은 시드로 같은 난수 시퀀스를 생성해야 한다. 그렇지 않으면 디싱크(desync, 동기화 어긋남)가 발생한다.

---

## 3. 가우시안 (정규) 분포

균등 분포 대신 종 모양의 정규 분포를 사용할 때가 많다.

### Box-Muller 변환

```text
function gaussian(mean, stddev):
    u1 = random()
    u2 = random()

    // 0 로그 방지
    u1 = max(u1, 1e-10)

    z0 = sqrt(-2 × ln(u1)) × cos(2π × u2)
    z1 = sqrt(-2 × ln(u1)) × sin(2π × u2)   // 두 번째 독립 표준정규 난수 (공짜)
    return mean + stddev × z0

// 기본: 평균 0, 표준편차 1
// 평균 μ, 표준편차 σ: mean + σ × z0
```

$$z_0 = \sqrt{-2 \ln(u_1)} \times \cos(2\pi \cdot u_2), \quad z_1 = \sqrt{-2 \ln(u_1)} \times \sin(2\pi \cdot u_2)$$

$$\text{result} = \mu + \sigma \cdot z_0$$

> **팁**: 한 번의 `ln`/`sqrt` 호출로 cos 항 $z_0$과 sin 항 $z_1$, 두 개의 독립적인 표준정규 난수를 얻는다. 필요하면 $z_1$도 캐싱해 재활용하자.

### 중심 극한 정리 활용 (간단 근사)

```text
// 12개 균등 난수의 합 ≈ 정규 분포
function gaussianApprox():
    sum = 0
    for i in 1..12:
        sum += random()
    return sum - 6    // 평균 0, 표준편차 1
```

$$z \approx \sum_{i=1}^{12} \text{random}_i - 6 \quad \text{평균 0, 표준편차 1}$$

**게임에서의 활용:**
- **데미지 변동**: 기본 데미지 ± 표준편차
- **AI 행동**: 같은 상황에서도 약간씩 다른 행동
- **스탯 분포**: 캐릭터 능력치의 자연스러운 분포
- **자연 현상**: 풀, 돌 등의 크기 분산

$$\text{damage} = \text{baseDamage} \times \text{gaussian}(1.0, 0.1) \quad \text{±10% 변동}$$

---

## 4. 분포 종류

### 균등 분포 (Uniform)

> 모든 값이 같은 확률. $[0, 1]$에서 0.1도 0.5도 0.9도 같은 확률

### 정규 분포 (Normal/Gaussian)

> 종 모양, 중심(평균) 근처가 가장 높은 확률. 극단값은 드묾

### 지수 분포 (Exponential)

$$r = \frac{-\ln(1 - \text{random()})}{\lambda} \quad \lambda = \text{이벤트 발생률}$$

### 삼각 분포 (Triangular)

> 최소 $a$, 최대 $b$, 최빈값 $c$ 지정

$$u = \text{random()}$$

$$\text{result} = \begin{cases} a + \sqrt{u \cdot (b - a)(c - a)} & \text{if } u < \frac{c - a}{b - a} \\ b - \sqrt{(1 - u) \cdot (b - a)(b - c)} & \text{otherwise} \end{cases}$$

---

## 5. 가중 난수 (Weighted Random)

각 항목이 다른 확률로 선택되어야 할 때 사용한다.

### 누적 확률 방식

```text
items = [
    { name: "common",    weight: 60 },
    { name: "rare",      weight: 30 },
    { name: "legendary", weight: 10 }
]
totalWeight = 100

function weightedRandom():
    r = random() × totalWeight
    cumulative = 0
    for item in items:
        cumulative += item.weight
        if r < cumulative:
            return item.name
```

### Alias Method (O(1) 선택)

```text
// 전처리 O(n), 선택 O(1)
// 항목이 많고 여러 번 뽑아야 할 때 최적

// 전처리: 각 항목에 alias 배정
// 선택: 난수 하나로 즉시 결정
```

**게임에서의 활용:**
- **가챠/뽑기**: 희귀도별 확률
- **아이템 드롭**: 적이 죽을 때 드롭 아이템 결정
- **맵 생성**: 어떤 지형이 생성될지
- **AI 행동 선택**: 행동마다 가중치 부여

```text
// 적 드롭 테이블
dropTable = [
    { item: "gold",    weight: 50 },
    { item: "potion",  weight: 30 },
    { item: "sword",   weight: 15 },
    { item: "artifact", weight: 5  }
]
drop = weightedRandom(dropTable)
```

---

## 6. 셔플 (Shuffle)

배열을 무작위로 섞는다.

### Fisher-Yates 셔플

```text
function shuffle(arr):
    for i from arr.length - 1 down to 1:
        j = floor(random() × (i + 1))    // [0, i]
        swap(arr[i], arr[j])
    return arr
```

> **주의**: 잘못된 셔플 (예: 각 원소를 random 위치로 보내기)은 편향이 생긴다. Fisher-Yates는 모든 순열이 같은 확률로 나오도록 보장한다.

**게임에서의 활용:**
- **카드 덱**: 카드 섞기
- **맵 요소**: 아이템 위치 무작위 배치
- **음악 재생**: 셔플 플레이리스트
- **퀘스트 순서**: 매번 다른 순서로 진행

---

## 7. 난수 생성기 (PRNG) 종류

| PRNG | 속도 | 주기 | 품질 | 게임 추천 |
|------|------|------|------|----------|
| LCG (선형 합동) | 빠름 | $2^{32}$ | 낮음 | 단순 게임 |
| Mersenne Twister | 보통 | $2^{19937}$ | 높음 | 일반적 |
| Xorshift | 매우 빠름 | $2^{128}$ | 보통 | 실시간 게임 |
| PCG | 빠름 | $2^{64}$ | 높음 | 현대 게임 |
| Halton/Sobol | 느림 | — | 낮음(편향) | **안 됨** |

> **실전 팁**: 게임에서는 PCG(Permuted Congruential Generator)를 추천한다. 빠르고 품질이 좋으며, 시드 재현이 완벽하다.

### Xorshift 예시

```text
state = seed    // 0이 아니어야 함

function xorshift():
    x = state
    x ^= x << 13
    x ^= x >> 17
    x ^= x << 5
    state = x
    return x / 0x100000000   // x / 2^32 → 최대 1 - 2^-32 < 1, [0, 1) 보장
```

---

## 8. 2D 난수 (원 내부)

```text
// 거부 샘플링 (간단하지만 비효율적)
function randomInCircle(radius):
    while true:
        x = random() × 2 - 1    // [-1, 1)
        y = random() × 2 - 1
        if x² + y² ≤ 1:
            return (x × radius, y × radius)

// 극좌표 (효율적)
function randomInCirclePolar(radius):
    angle = random() × 2π
    r = radius × sqrt(random())    // sqrt로 균등 분포
    return (r × cos(angle), r × sin(angle))
```

$$\text{angle} = \text{random()} \times 2\pi$$

$$r = \text{radius} \times \sqrt{\text{random()}}$$

$$\text{position} = (r \cdot \cos(\text{angle}),\ r \cdot \sin(\text{angle}))$$

> **주의**: $r = \text{random()} \times \text{radius}$는 중심에 밀집하는 편향이 생긴다. $\sqrt{\text{random()}}$을 해야 균등하다.

---

## 9. 빠른 참조

| 분포 | 공식 | 활용 |
|------|------|------|
| 균등 | $\text{random()}$ | 기본 |
| 정규 | Box-Muller | 데미지, 스탯 |
| 지수 | $\frac{-\ln(1-r)}{\lambda}$ | 대기 시간 |
| 삼각 | $a + \sqrt{u(b-a)(c-a)}$ | 제한된 범위 |
| 가중 | 누적 확률 | 드롭, 가챠 |
| 셔플 | Fisher-Yates | 카드, 순서 |

| 팁 | 설명 |
|----|------|
| 시드 고정 | 재현 가능 |
| PCG | 현대 추천 PRNG |
| 극좌표 $\sqrt{}$ | 원 내 균등 |
| Alias method | $O(1)$ 가중 선택 |