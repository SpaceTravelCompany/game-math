---
title: 노이즈 함수 (Noise Functions)
slug: noise
---

## 소개

노이즈 함수는 무작위이면서도 자연스러운 패턴을 생성하는 수학 함수다. 지형 생성, 구름, 질감, 물결, 안개, 입자 효과 등 게임의 프로시저럴 콘텐츠 생성에 필수적이다.

> **핵심**
> 노이즈는 랜덤과 다르다. 랜덤은 각 점이 독립이지만, 노이즈는 인접한 점이 부드럽게 변한다.

---

## 1. 랜덤 vs 노이즈

```text
// 랜덤: 각 픽셀이 독립적 → 픽셀 노이즈 (TV 지지직거림)
value = random(x, y)    // 이웃과 전혀 무관

// 노이즈: 이웃 값이 부드럽게 연결 → 자연스러운 패턴
value = noise(x, y)     // 인접 픽셀이 비슷한 값
```

```text
랜덤:  █▓░██▒▓░▒█▓░██▒    // 각 픽셀 무작위
노이즈: ▓▒▒░░▒▓▓▒▒░░▒▓    // 부드러운 전환
```

---

## 2. Value Noise

가장 단순한 노이즈. 격자점에 랜덤 값을 배치하고 보간한다.

### 알고리즘

```text
// 1. 격자점에 랜덤 값 배정
grid[x][y] = random(x, y)    // 정수 좌표에서만 랜덤

// 2. 실수 좌표에서 보간
function valueNoise(x, y):
    x0 = floor(x), x1 = x0 + 1
    y0 = floor(y), y1 = y0 + 1

    fx = smoothstep(x - x0)    // 부드러운 보간
    fy = smoothstep(y - y0)

    v00 = grid[x0][y0]
    v10 = grid[x1][y0]
    v01 = grid[x0][y1]
    v11 = grid[x1][y1]

    // 이중 선형 보간
    top = lerp(v00, v10, fx)
    bottom = lerp(v01, v11, fx)
    return lerp(top, bottom, fy)
```

### 특징

- 구현이 간단
- 격자점이 약간 보일 수 있음 (블로키)
- 품질이 낮지만 빠름

---

## 3. Perlin Noise

Ken Perlin이 개발한 노이즈. 격자점에 그라디언트(방향 벡터)를 배치하여 더 자연스러운 결과를 만든다.

### 알고리즘

```text
function perlin(x, y):
    // 1. 격자 좌표
    x0 = floor(x), x1 = x0 + 1
    y0 = floor(y), y1 = y0 + 1

    // 2. 로컬 좌표
    sx = fade(x - x0)
    sy = fade(y - y0)

    // 3. 각 격자점의 그라디언트와 로컬 벡터의 내적
    n00 = dot(gridGradient(x0, y0), (x-x0, y-y0))
    n10 = dot(gridGradient(x1, y0), (x-x1, y-y0))
    n01 = dot(gridGradient(x0, y1), (x-x0, y-y1))
    n11 = dot(gridGradient(x1, y1), (x-x1, y-y1))

    // 4. 보간
    return lerp(
        lerp(n00, n10, sx),
        lerp(n01, n11, sx),
        sy
    )

// fade 함수 (6t⁵ - 15t⁴ + 10t³)
function fade(t):
    return t³ × (t × (t × 6 - 15) + 10)
```

$$\text{fade}(t) = 6t^5 - 15t^4 + 10t^3$$

### 특징

- 자연스러운 결과 (구름, 지형, 질감)
- 격자점이 덜 보임
- 출력 범위: 약 $[-1, 1]$
- Value Noise보다 계산이 약간 복잡

**게임에서의 활용:**
- **지형 생성**: 하이트맵 생성
- **구름**: 하늘의 구름 패턴
- **질감**: 목재, 대리석, 돌
- **물면**: 수면 파동
- **동굴**: 3D Perlin으로 동굴 시스템

### 타일링 / 주기 노이즈 (Periodic Noise)

노이즈를 타일맵이나 반복 텍스처에 쓰려면 가장자리가 이음새 없이 이어져야 한다. 격자점 해시를 주기 $p$로 모듈러 연산하면 된다:

```text
// 주기 p로 타일링되는 격자 해시
function hashTiled(x, y, p):
    xi = x mod p        // ← p 주기로 좌표를 접음
    yi = y mod p
    return perm[perm[xi] + yi]
```

- **Perlin**: permutation 테이블(0~255)을 그대로 쓰면 본래 $256$ 주기(각 축)로 타일링된다. 임의 주기 $p$가 필요하면 인덱스를 `x mod p`로 접는다.
- **Simplex**: 비정형(단체) 격자라 격자점을 규칙적으로 접을 수 없어, 좌표를 변환한 뒤 타일링하는 별도 처리가 필요하다.

---

## 4. Simplex Noise

Perlin을 개선한 버전. 정사각형 격자 대신 정삼각형 격자를 사용한다.

### 개선점

```text
// Perlin: 정사각형 격자 (2D: 4점, 3D: 8점, 4D: 16점)
// Simplex: 단체(simplex) 격자 (2D: 3점, 3D: 4점, 4D: 5점)

// → 차원이 높아질수록 Simplex가 훨씬 빠름
// → 방향성 편향이 적음
// → 더 균등한 결과
```

### 특징

| 특징 | Perlin | Simplex |
|------|--------|---------|
| 격자 | 정사각형 | 단체(Simplex) |
| 속도 (2D) | 비슷 | 비슷 |
| 속도 (3D+) | 느림 | 빠름 |
| 복잡도 ($n$차원) | $O(n\,2^n)$ — 격자점 $2^n$개 | $O(n^2)$ — 단체 꼭짓점 $n+1$개 |
| 편향 | 있음 | 적음 |
| 특허 | 없음 (알고리즘에 특허 미출원) | US 6,867,776 — 만료 (2022-01-08) |
| 구현 | 쉬움 | 복잡 |

> **실전 팁**: 2D에서는 Perlin으로 충분. 3D 이상에서는 Simplex가 더 빠르고 품질이 좋다. 최신 엔진들은 대부분 Simplex를 사용한다.

---

## 5. Fractal Brownian Motion (fBm)

여러 옥타브(octave)의 노이즈를 중첩하여 더 복잡하고 자연스러운 패턴을 만든다.

### 알고리즘

```text
function fbm(x, y, octaves, persistence, lacunarity):
    total = 0
    amplitude = 1
    frequency = 1
    maxValue = 0    // 정규화용

    for i in 0..octaves:
        total += noise(x × frequency, y × frequency) × amplitude
        maxValue += amplitude
        amplitude *= persistence     // 0.5 일반적
        frequency *= lacunarity       // 2.0 일반적

    return total / maxValue    // [-1, 1] 정규화
```

$$\text{fbm}(x, y) = \frac{1}{\text{maxValue}} \sum_{i=0}^{\text{octaves}} \text{noise}(x \cdot f_i) \cdot a_i$$

$$a_i = \text{persistence}^i, \quad f_i = \text{lacunarity}^i$$

### 파라미터 설명

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| octaves | 4-8 | 중첩 횟수 (클수록 디테일) |
| persistence | 0.5 | 각 옥타브의 진폭 감소율 |
| lacunarity | 2.0 | 각 옥타브의 주파수 증가율 |

### 옥타브의 의미

```text
// 낮은 주파수 → 큰 형태 (산맥)
// 높은 주파수 → 세부 디테일 (돌, 잔해)

octave 0: 큰 산맥
octave 1: 중간 언덕
octave 2: 작은 요철
octave 3: 미세 질감
...
```

**게임에서의 활용:**
- **지형**: 산맥 + 언덕 + 자갈의 자연스러운 중첩
- **구름**: 큰 구름 + 작은 구름 조직
- **질감**: 나뭇결의 큰 패턴 + 미세한 섬유
- **해면**: 큰 파도 + 잔물결

---

## 6. 3D 노이즈와 볼륨 렌더링

```text
// 3D Perlin/Simplex 노이즈
density = noise3D(x, y, z)

// 구름 렌더링
cloudDensity = fbm3D(x, y, z, 5, 0.5, 2.0)

// 동굴 시스템 (3D 임계값)
threshold = 0.5
if noise3D(x, y, z) > threshold:
    solid = true     // 땅
else:
    solid = false    // 공기 (동굴)
```

### 2D 지형 vs 3D 볼륨

```text
// 2D: 하이트맵 (높이만 변화)
height = fbm2D(x, z) × maxHeight
terrain(x, y, z) = y < height ? solid : air

// 3D: 밀도 필드 (오버hang, 동굴 가능)
density = fbm3D(x, y, z)
terrain = density > threshold ? solid : air
// → 동굴, 오버hang, 3D 구조 가능
```

---

## 7. 도메인 워핑 (Domain Warping)

노이즈의 입력 좌표를 다른 노이즈로 변형하여 더 복잡한 패턴을 만든다.

```text
// 원본 노이즈
value = noise(x, y)

// 워핑된 노이즈
wx = x + noise(x, y) × warpStrength
wy = y + noise(x + 100, y + 100) × warpStrength
value = noise(wx, wy)
```

### 다중 워핑

```text
// 더 복잡한 패턴
q = vec2(noise(x, y), noise(x + 5.2, y + 1.3))
r = vec2(noise(x + 4*q.x + 1.7, y + 4*q.y + 9.2),
         noise(x + 4*q.x + 8.3, y + 4*q.y + 2.8))
final = noise(x + 4*r.x, y + 4*r.y)
```

**게임에서의 활용:**
- **대리석 질감**: 휘어진 결 패턴
- **용암**: 흐르는 듯한 왜곡
- **구름**: 더 자연스러운 구름 형태
- **지도**: 대륙 형태의 자연스러운 해안선

---

## 8. 노이즈 최적화

### 캐싱

```text
// 하이트맵을 미리 계산하여 텍스처로 저장
heightmap = generateNoiseTexture(width, height, seed)

// 런타임에는 텍스처 샘플링만
height = sampleTexture(heightmap, uv)
```

### LOD (Level of Detail)

```text
// 거리에 따라 옥타브 수 조절
if distance > 100:
    octaves = 2    // 멀면 단순
elif distance > 50:
    octaves = 4
else:
    octaves = 8    // 가까우면 디테일
```

### GPU 셰이더

```text
// 픽셀 셰이더에서 노이즈 계산
// 각 픽셀이 병렬로 처리 → 매우 빠름
// GLSL Perlin/Simplex 구현 사용
```

---

## 9. 실전 예시: 지형 생성

```text
function generateTerrain(width, height, seed):
    noise.setSeed(seed)

    for x in 0..width:
        for z in 0..height:
            // fBm으로 높이 계산
            h = fbm(x × 0.01, z × 0.01, 6, 0.5, 2.0)

            // 높이 범위 매핑
            height = (h + 1) × 0.5 × maxHeight

            // 생물군계 결정
            moisture = fbm(x × 0.005 + 1000, z × 0.005 + 1000, 4, 0.5, 2.0)

            if height < waterLevel:
                biome = "ocean"
            elif height < waterLevel + 5:
                biome = "beach"
            elif moisture > 0.5:
                biome = "forest"
            else:
                biome = "plains"

            terrain[x][z] = { height, biome }
```

---

## 10. 실전 예시: 안개/연기 효과

```text
// 파티클 위치를 노이즈로 변형하여 자연스러운 연기
function updateSmoke(particle, time):
    n = noise3D(particle.x × 0.1, particle.y × 0.1, time × 0.5)

    // 노이즈로 속도 변형
    particle.vx += n × 0.5 × dt
    particle.vy += 1.0 × dt         // 상승

    // 감쇠
    particle.vx *= 0.95
    particle.vy *= 0.98

    particle.x += particle.vx × dt
    particle.y += particle.vy × dt
    particle.opacity -= dt × 0.3
```

---

## 11. 빠른 참조

| 노이즈 | 품질 | 속도 | 용도 |
|--------|------|------|------|
| Value | 낮음 | 빠름 | 단순 패턴 |
| Perlin | 높음 | 보통 | 지형, 질감 |
| Simplex | 높음 | 빠름(3D+) | 현대 게임 |
| fBm | 매우 높음 | 느림 | 자연 현상 |
| Domain Warp | 복잡 | 느림 | 대리석, 용암 |

| 파라미터 | 기본값 | 효과 |
|----------|--------|------|
| octaves | 4-8 | 디테일 양 |
| persistence | 0.5 | 진폭 감소 |
| lacunarity | 2.0 | 주파수 증가 |
| scale | 0.01 | 노이즈 크기 |
| threshold | 0 | 솔리드/에어 경계 |