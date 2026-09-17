---
title: 바리센트릭 좌표 (Barycentric Coordinates)
slug: barycentric
---

## 소개

바리센트릭(무게중심) 좌표는 삼각형(또는 더 일반적인 단체) 내 한 점을 꼭지점들의 가중합으로 표현하는 좌표계다. 《거리 & 최단점》 §6(점-삼각형 거리)에서 이미 점이 삼각형 내부인지 판별하는 데 사용되었다.

> **핵심**
> 바리센트릭 좌표 $(u,v,w)$는 삼각형 위 어느 점이든 세 꼭지점의 가중 평균으로 나타낸다. $u+v+w=1$이고 모두 $0$ 이상이면 점은 삼각형 내부에 있다.

---

## 1. 정의

삼각형 $\triangle \mathbf{ABC}$ 위의 점 $\mathbf{P}$는 세 꼭지점의 가중합으로 표현된다:

$$
\mathbf{P} = u\mathbf{A} + v\mathbf{B} + w\mathbf{C}, \qquad u + v + w = 1
$$

$(u, v, w)$를 **바리센트릭 좌표**라고 부른다.

```text
          C
         /\
        /  \
       /    \
      /      \
     /   .P   \
    /          \
   A——————-----B

   P = u·A + v·B + w·C
   u = area(PBC) / area(ABC)
   v = area(APC) / area(ABC)
   w = area(ABP) / area(ABC)
```

### 위치 판정

| 조건 | 의미 |
|------|------|
| $u,v,w \ge 0$ | 삼각형 **내부** |
| 하나가 $0$ | 변 위 |
| 둘이 $0$ | 꼭지점 |
| 하나가 $< 0$ | 삼각형 **외부** |

### 이름의 유래

"바리센터"(무게중심)라는 이름은 물리적 비유에서 왔다. 세 꼭지점에 각각 무게 $u,v,w$를 달았을 때 전체 계의 무게중심이 정확히 $\mathbf{P}$에 위치한다. $u=v=w=\frac13$이면 기하학적 무게중심(centroid)이다.

---

## 2. 계산법

### 2D에서 — 면적 비율

2D에서는 넓이 비율로 직접 구한다:

$$
u = \frac{\text{area}(\triangle \mathbf{PBC})}{\text{area}(\triangle \mathbf{ABC})},
\quad
v = \frac{\text{area}(\triangle \mathbf{APC})}{\text{area}(\triangle \mathbf{ABC})},
\quad
w = \frac{\text{area}(\triangle \mathbf{ABP})}{\text{area}(\triangle \mathbf{ABC})}
$$

```text
// 2D 바리센트릭 (면적 비율)
function barycentric2D(P, A, B, C):
    areaABC = cross(B - A, C - A)    // 평행사변형 넓이 = 2×삼각형

    u = cross(B - P, C - P) / areaABC
    v = cross(C - P, A - P) / areaABC
    w = cross(A - P, B - P) / areaABC

    return (u, v, w)
```

> **참고**: $\text{cross}$는 2D 외적 $a_x b_y - a_y b_x$. 분모인 `areaABC`가 $0$이면 삼각형이 퇴화(degenerate)한 것이므로 미리 검사한다.

### 3D에서 — 투영 후 풀이

3D에서는 삼각형이 놓인 평면에 좌표계를 만들어 푼다. 《거리 & 최단점》 §6 코드가 이 방식을 사용한다:

```text
// 3D 바리센트릭 (거리 & 최단점 §6 방식)
function barycentric3D(P, A, B, C):
    v0 = B - A
    v1 = C - A
    v2 = P - A

    d00 = dot(v0, v0)
    d01 = dot(v0, v1)
    d11 = dot(v1, v1)
    d20 = dot(v2, v0)
    d21 = dot(v2, v1)

    denom = d00 * d11 - d01 * d01
    v = (d11 * d20 - d01 * d21) / denom
    w = (d00 * d21 - d01 * d20) / denom
    u = 1 - v - w

    return (u, v, w)
```

이는 선형 시스템 $\begin{bmatrix}\mathbf{B}-\mathbf{A} & \mathbf{C}-\mathbf{A}\end{bmatrix} \begin{bmatrix}v \\ w\end{bmatrix} = \mathbf{P} - \mathbf{A}$를 최소제곱으로 푼 것과 동일하다.

> **주의**: $\text{denom}=0$이면 삼각형이 퇴화된 경우다. 면이 거의 0에 가까운 삼각형(needle triangle)에서는 수치 오차가 커질 수 있으므로 임계값 $\varepsilon$으로 검사한다.

---

## 3. 보간 (Interpolation)

바리센트릭 좌표의 가장 강력한 용도는 꼭지점 속성을 삼각형 표면 위에서 **부드럽게 보간**하는 것이다.

```text
// 꼭지점 속성 보간 (§1 관례: u→A, v→B, w=1-u-v→C)
w = 1 - u - v

// 법선 보간 (부드러운 셰이딩)
normal = u * N_A + v * N_B + w * N_C
normal = normalize(normal)

// UV 좌표 보간
uv = u * UV_A + v * UV_B + w * UV_C

// 색상 보간
color = u * C_A + v * C_B + w * C_C
```

### 보간 공식

속성 $\mathbf{f}$가 각 꼭지점에서 $\mathbf{f}_A, \mathbf{f}_B, \mathbf{f}_C$일 때:

$$
\mathbf{f}(\mathbf{P}) = u\,\mathbf{f}_A + v\,\mathbf{f}_B + w\,\mathbf{f}_C
$$

```text
     C (UV=0,1)    ← 텍스처 좌표 예시
    /\
   /  \
  /    \
 /  .P  \         ← P에서의 UV = (0.33, 0.33)
/        \
A——————--B
(0,0)    (1,0)
```

바리센트릭 보간은 3D 공간의 평면 위에서 완벽한 **선형 보간**이다. 이 덕분에:
- **퐁 셰이딩(Phong shading)**: 각 픽셀에서 법선을 보간해 부드러운 광택 표현
- **텍스처 매핑**: UV 좌표 보간으로 삼각형 위에 텍스처 입히기
- **높이맵**: 꼭지점 높이값 보간으로 지형 표면 생성

### 원근 교정 보간 (Perspective-Correct Interpolation)

3D 장면을 2D 화면에 원근 투영하면 원근감 때문에 화면 상의 거리가 균일하지 않다.  
화면의 2D 픽셀 공간에서 바리센트릭 좌표로 $(u, v, w)$ 속성을 그대로 선형 보간하면 **원근 왜곡(Perspective Warp)**이 생겨 텍스처가 지그재그로 뒤틀린다(초기 플레이스테이션 1 게임들의 텍스처 흔들림 현상).

현대 GPU는 이를 해결하기 위해 동차 좌표 깊이 $w$를 이용해 **원근 교정 보간**을 수행한다:

$$\frac{\mathbf{f}(\mathbf{P})}{w} = u\,\frac{\mathbf{f}_A}{w_A} + v\,\frac{\mathbf{f}_B}{w_B} + w\,\frac{\mathbf{f}_C}{w_C}$$

$$\frac{1}{w} = u\,\frac{1}{w_A} + v\,\frac{1}{w_B} + w\,\frac{1}{w_C}$$

$$\mathbf{f}_{\text{correct}} = \frac{\mathbf{f}(\mathbf{P}) / w}{1 / w}$$

> GPU의 래스터라이저는 각 픽셀마다 속성을 $1/w$로 나누어 보간한 후, 보간된 $1/w$로 다시 나누어 원근 왜곡 없는 완벽한 텍스처 매핑을 구현한다.

---

## 4. 게임 활용

### 지형/메시 픽킹

마우스 클릭으로 지형의 어느 지점을 선택했는지 알아내려면 광선-삼각형 교차(Möller–Trumbore, 《광선 교차》 §5)로 $u,v$를 얻고 바리센트릭 보간으로 UV/법선을 추출한다:

```text
// 마우스 픽킹 — 교차점의 법선과 UV 얻기
hit, t, u, v = rayTriangle(ray, V0, V1, V2)

if hit:
    // 여기 u, v는 rayTriangle(Möller–Trumbore) 출력으로 V1, V2에 대응 (w = 1-u-v → V0)
    w = 1 - u - v
    normal = w * N0 + u * N1 + v * N2
    uv = w * UV0 + u * UV1 + v * UV2
    // UV로 텍스처 색상 샘플링, 법선으로 조명 계산
```

### 스킨드 메시 가중치

캐릭터 스키닝에서 각 정점은 여러 본(bone)에 바리센트릭과 유사한 방식으로 가중치가 할당된다:

```text
// 스키닝 — 4개 본 가중치 (일반화된 바리센트릭)
position = w0 * boneMat[0] * bindPos +
           w1 * boneMat[1] * bindPos +
           w2 * boneMat[2] * bindPos +
           w3 * boneMat[3] * bindPos
// Σ wi = 1, wi ≥ 0
```

### 포인트 인 폴리곤

볼록 다각형은 여러 삼각형으로 분할(tessellation)한 뒤 각 삼각형에 대해 바리센트릭 내부 판정을 수행한다. 오목 다각형은 이 방식으로 직접 처리할 수 없으며 별도의 알고리즘(레이 캐스팅, winding number)이 필요하다.

### 지형 충돌

캐릭터가 지형 메시 위에 서 있을 때, 발 아래 삼각형의 바리센트릭 좌표로 정확한 높이를 보간한다:

```text
// 삼각형 위 높이 보간 (§1 관례: u→A, v→B, w→C)
height = u * h_A + v * h_B + w * h_C
character.position.y = height
```

---

## 5. 빠른 참조

| 항목 | 수식/조건 | 설명 |
|------|-----------|------|
| 정의 | $\mathbf{P} = u\mathbf{A} + v\mathbf{B} + w\mathbf{C}$ | 꼭지점 가중합 |
| 제약 | $u + v + w = 1$ | 무게합 1 |
| 내부 조건 | $u \ge 0,\; v \ge 0,\; w \ge 0$ | 모두 0 이상이면 내부 |
| 무게중심 | $u = v = w = 1/3$ | centroid |
| 넓이 비율 (2D) | $u = \text{area}(\triangle \text{PBC}) / \text{area}(\triangle \text{ABC})$ | 2D 직접 계산 |
| 3D 계산 | 선형 시스템 풀이 | 거리 & 최단점 §6 |
| 속성 보간 | $\mathbf{f}(\mathbf{P}) = u\mathbf{f}_A + v\mathbf{f}_B + w\mathbf{f}_C$ | 법선/UV/색상 |
| 교차 참조 | 점-삼각형 (§6) | 《거리 & 최단점》 §6 |
| 교차 참조 | 법선 보간 | 《광선 교차》 §5 |
| 교차 참조 | 점-삼각형 내부 판별 | 《외적》 §8 |
