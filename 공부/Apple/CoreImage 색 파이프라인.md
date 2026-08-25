---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-25
updated: 2026-08-25
projects:
  - "[[프로젝트/개인/즉석카메라/README|즉석카메라]]"
---

# CoreImage 색 파이프라인

필름 룩·현상 연출 같은 **시간에 따라 변하는 색 보정**을 CoreImage로 만들 때의 원칙과 함정. 즉석카메라 앱의 10분 현상 곡선을 만들며 정리했다.

## 핵심 정리

### 1. 색조 이동은 생각보다 훨씬 약해야 한다 ★

"차갑게" 만들려고 채널 게인을 크게 건드리면 **색조가 통째로 무너진다.**

```swift
// ❌ 전체가 새파래진다
"inputRVector": CIVector(x: 0.45, ...)   // R을 절반 이하로
"inputBVector": CIVector(x: 0, y: 0, z: 1.30, w: 0)

// ✅ "차갑다"는 인상은 주면서 색조는 유지
"inputRVector": CIVector(x: 0.87, ...)   // −13%
"inputGVector": CIVector(x: 0, y: 0.97, ...)  // −3%
"inputBVector": CIVector(x: 0, y: 0, z: 1.07, w: 0)  // +7%
```

**필름의 청록빛은 눈에 띄는 색이 아니라 미묘한 편향이다.** ±10% 안쪽에서 논다.

### 2. 블랙 리프트는 `bias`가 아니라 톤커브로

`CIColorMatrix`의 `inputBiasVector`로 섀도를 들면 **전체가 탁해진다**(모든 픽셀에 상수를 더하므로 하이라이트도 같이 뜬다).

**`CIToneCurve`로 S커브를 그리는 게 맞다** — 섀도만 들고 하이라이트는 굴린다.

```swift
o.applyingFilter("CIToneCurve", parameters: [
    "inputPoint0": CIVector(x: 0.00, y: 0.045),  // 섀도 리프트
    "inputPoint1": CIVector(x: 0.25, y: 0.24),
    "inputPoint2": CIVector(x: 0.50, y: 0.52),
    "inputPoint3": CIVector(x: 0.75, y: 0.80),
    "inputPoint4": CIVector(x: 1.00, y: 0.98)])  // 하이라이트 롤오프
```

`bias`는 **그림자에만 색을 얹을 때**(R 0.004 / G 0.012 / B 0.020처럼 아주 작은 값) 쓰는 게 맞다.

### 3. 시간 연출은 "여러 속성이 서로 다른 속도로" 움직여야 자연스럽다

하나의 값으로 페이드하면 가짜처럼 보인다. 폴라로이드 현상을 재현할 때 실제로 쓴 시차:

| 속성 | 구간(t) | 지수 |
|---|---|---|
| 흰 베일 | 0 ~ 0.55 | 0.75 (가장 먼저) |
| 밝기 | 0 ~ 0.65 | 0.9 |
| 대비 | 0.08 ~ 0.80 | 1.2 |
| **채도** | 0.12 ~ 0.95 | **1.5 (가장 늦게)** |
| 색온도 | 0.20 ~ 0.92 | 1.4 |

구간·지수를 분리하는 헬퍼 하나면 된다:

```swift
func ramp(_ t: Double, _ a: Double, _ b: Double, _ p: Double = 1) -> Double {
    pow(clamp((t - a) / (b - a)), p)   // a 이전 0, b 이후 1
}
```

**"형태 먼저, 색 나중"**이 핵심 인상이다 — 채도를 가장 늦게 올리면 중간에 **흑백 사진처럼 형태만 보이는 상태**가 만들어진다.

### 3-b. 스플릿 톤은 `CIColorPolynomial` — `CIToneCurve`로는 안 된다 ★

`CIToneCurve`는 **휘도 기준**이라 채널별로 다른 커브를 못 건다. "그림자는 청록, 하이라이트는 크림" 같은 **스플릿 톤**을 만들려면 `CIColorPolynomial`로 채널마다 따로 커브를 준다.

```swift
// out = a + (b−a)·[(1−k)x + 3k·x² − 2k·x³]   (a=그림자, b=하이라이트, k=S커브 강도)
func coef(_ a: Double, _ b: Double, _ k: Double) -> CIVector {
    let d = b - a
    return CIVector(x: a, y: d*(1-k), z: d*3*k, w: -d*2*k)   // c0,c1,c2,c3
}
img.applyingFilter("CIColorPolynomial", parameters: [
    "inputRedCoefficients":   coef(0.100, 0.985, 0.14),
    "inputGreenCoefficients": coef(0.116, 0.958, 0.14),
    "inputBlueCoefficients":  coef(0.148, 0.888, 0.14)])   // B가 그림자↑ 하이라이트↓
```

**B의 그림자 리프트가 가장 높고 하이라이트 상한이 가장 낮은 것**이 폴라로이드 룩의 정체다.

### 3-c. 필름 톤의 효과는 생각보다 세게 걸어야 보인다

그림자 리프트를 0.03~0.07로 잡았더니 **변주 다섯 개가 육안으로 구분되지 않았다.** 실제 필름 룩은 **0.10~0.15**다. 그리고 **어둡고 차분한 사진에서는 톤 차이가 묻히므로 밝은 사진으로도 반드시 교차 확인**한다 — 톤은 사진에 따라 전혀 다르게 먹는다.

### 4. 반투명 레이어 합성은 `CIColor`의 alpha로

`CIDissolveTransition`을 쓰면 코드가 지저분해지고 extent 관리가 번거롭다. **색 자체에 alpha를 넣고 `CISourceOverCompositing`이 훨씬 간단하다.**

```swift
let veil = CIImage(color: CIColor(red: 0.93, green: 0.935, blue: 0.915, alpha: amount))
             .cropped(to: base.extent)
let out = veil.applyingFilter("CISourceOverCompositing",
                              parameters: [kCIInputBackgroundImageKey: img])
```

`CIImage(color:)`는 **무한 extent**라 반드시 `cropped(to:)`를 걸어야 한다.

### 5. 색은 **맥락 없이 판정하면 안 된다** ★

같은 픽셀이 배경과 프레임에 따라 전혀 다르게 읽힌다. 즉석카메라 톤을 이미지 단독으로 봤을 때는 *"원본보다 안 예쁘다"*였는데, **흰 인화지 테두리 안에 넣고 어두운 배경에 올리자 훨씬 좋아 보였다.** 판정을 뒤집을 만큼 차이가 컸다.

→ **색 작업의 검증은 최종 UI 맥락(프레임·배경)을 넣은 상태에서 한다.**

### 6. 검증은 앱이 아니라 PNG 스트립으로

Xcode 프로젝트·시뮬레이터 없이 `swiftc`로 렌더 스크립트를 묶어 **시간대별 프레임을 한 장의 스트립으로** 뽑는다. 한 사이클 30초. [[프로젝트/개인/Zappy/README|Zappy]] 테마 검증에 쓰던 방식이 색 파이프라인에도 그대로 통한다.

`NSGraphicsContext`로 스트립을 조립할 때 `CGFloat`/`Int` 혼용이 잦은데, **좌표 계산을 미리 `CGFloat` 지역변수로 빼두지 않으면** Swift 타입 체커가 *"unable to type-check this expression in reasonable time"*으로 죽는다.

## 기록

### 2026-08-25 — 즉석카메라 10분 현상 곡선

- 맥락: [[프로젝트/개인/즉석카메라/README|즉석카메라]] 마일스톤 0. 앱을 만들기 전에 현상 연출만 떼어내 검증했다.
- 배운 것: 위 6개 항목 전부. 특히 ① **색조 이동의 적정 폭(±10% 안쪽)** ② **블랙 리프트는 bias가 아니라 톤커브** ③ **색 판정에는 UI 맥락이 필요하다**.
- 근거: `(로컬 경로)` — 1차(R 0.45배)는 전 구간이 파랗게 나와 폐기, 2차(R −13%)에서 정상화. 상세는 [[프로젝트/개인/즉석카메라/즉석카메라 개발 기록 2026-08-25|개발 기록 2026-08-25]].

## 참고 자료

- [Core Image Filter Reference — CIToneCurve / CITemperatureAndTint / CIColorMatrix](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html) — 필터별 파라미터 정의 (2026-08-25 확인)
