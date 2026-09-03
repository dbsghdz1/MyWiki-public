---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-25
updated: 2026-09-02
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

### 2026-09-02 — 폴라로이드다움은 색이 아니라 **결함**이다 ★ (Fadeo 1.1)

- 맥락: [[프로젝트/개인/즉석카메라/README|Fadeo]] 1.1. 필름 4종을 6종으로 늘리려는데, 필름을 가르는 파라미터가 숫자 6개(그림자 3·하이라이트 3에 커브·채도·비네트)뿐이라 **늘려도 "같은 사진의 색만 조금 다른 것"이 늘 뿐이었다.**
- 배운 것: 색이 맞아도 실물로 안 읽히는 이유는 **필름이 화학이라서 생기는 결함이 하나도 없어서다.** 네 겹을 넣으니 필름끼리 색이 아니라 질감으로 갈렸다.

| 겹 | 정체 | 구현 |
|---|---|---|
| **할레이션** | 빛이 유제층을 지나 뒤에서 반사돼 돌아온다. 장파장이 가장 멀리 번져서 **붉다** | 하이라이트만 뽑아(`CIColorClamp` → `CIColorPolynomial`로 [threshold,1]→[0,1] 펴기) 붉게 물들이고 블러 → **스크린 블렌드** |
| **입자** | ISO 640 은염 알갱이 | `CIRandomGenerator` → 회색 → 블러 → 0.5 중심 진폭 → **`CIOverlayBlendMode`** (오버레이는 0.5가 무변화라 중간톤에 세게, 검정·흰색엔 거의 안 들어간다 — 은염 분포가 그렇다) |
| **번짐** | 현상액이 퍼지며 경계가 무뎌진다 | 블러 → **`CIMix`**(`kCIInputAmountKey`) |
| **현상 얼룩** | 롤러가 한쪽에서 밀어 생기는 좌우 비대칭 농도차 | `CILinearGradient` → `CISourceOverCompositing`. **완벽한 동심원 `CIVignette`만 있으면 필름이 아니라 인쇄물로 보인다** |

- **흑백은 순서가 전부다.** 채도를 **먼저** 0으로 죽이고 스플릿 톤을 얹어야 은염 인화의 "차가운 그림자, 따뜻한 하이라이트"가 나온다. 순서가 반대면 뒤따르는 채도 0이 방금 입힌 색을 지워서 그냥 색 빠진 사진이 된다.
- 근거: `(로컬 경로)`(6종 렌더)·`polaroid.swift`(겹별 기여도), 사진 3장 교차 검증. 앱 쪽은 `InstantCam` 커밋 `2c678bf`, `Sources/Film/ImagePipeline.swift`.

### 2026-09-02 — `CIAdditionCompositing`은 알파도 더한다 ★★ (Fadeo 1.1)

빛을 더하려고 가산 합성을 쓰면 **색이 절반으로 죽는다.** 알파 1짜리 두 장을 더하면 결과 알파가 2가 되고, CoreImage가 프리멀티플라이드를 되돌리는 순간 RGB가 그만큼 나눠진다.

증상이 단계적으로 나타나서 헷갈렸다. 겹을 하나씩 걸 때는 멀쩡한데 **전부 겹친 것만 새까맣게** 나왔다.

그렇다고 레이어 알파를 0으로 눌러 두면 이번엔 **그 레이어가 통째로 무시된다** — 채널 분리 후 합칠 때 R만 살아남아 화면이 새빨갛게 나왔다.

```swift
// ❌ 알파가 2가 되어 색이 죽는다
blurred.applyingFilter("CIAdditionCompositing", parameters: [kCIInputBackgroundImageKey: img])
// ❌ 알파 0 레이어는 삼켜진다
additive(gOnly).applyingFilter("CIAdditionCompositing", ...)   // G·B가 사라짐

// ✅ 빛을 더하는 자리 — 스크린 블렌드. 1-(1-a)(1-b), 알파를 안 건드린다
blurred.applyingFilter("CIScreenBlendMode", parameters: [kCIInputBackgroundImageKey: img])
// ✅ 두 그림을 비율로 섞는 자리
soft.applyingFilter("CIMix", parameters: [kCIInputBackgroundImageKey: img, kCIInputAmountKey: 0.25])
```

**규칙: CoreImage에서 여러 이미지를 합칠 때 `CIAdditionCompositing`은 쓰지 않는다.** 빛은 스크린, 혼합은 `CIMix`.

### 2026-09-02 — `CIRandomGenerator`는 **알파까지 난수**다 ★ (Fadeo 1.1)

필름 입자를 넣었더니 소금을 뿌린 것처럼 **흰 점이 박혔다.** 진폭을 0.42 → 0.085로 줄여도, 알갱이를 키우고 블러로 뭉쳐도 그대로였다.

원인은 진폭이 아니었다. `CIRandomGenerator`는 RGB뿐 아니라 **알파도 0~1 난수로 채운다.** CoreImage가 그걸 프리멀티플라이드로 해석하면서 알파가 0에 가까운 픽셀의 색이 1을 훌쩍 넘는 값으로 튀고, 렌더할 때 클램프되어 흰 점이 된다.

계측(400×400, R 채널):

| 단계 | min | max | **avg** |
|---|---|---|---|
| 원본 | 0 | 255 | 127.4 |
| 회색 + 알파 1 | 0 | 255 | **190.8** |
| + 블러 0.9 | 120 | 255 | **253.5** ← 거의 흰색 |
| 알파를 **먼저** 1로 클램프한 뒤 진폭 0.085 | 123 | 138 | 132.9 ✅ |

```swift
CIFilter(name: "CIRandomGenerator")!.outputImage!
    .applyingFilter("CIColorClamp", parameters: [          // ← 이게 먼저
        "inputMinComponents": CIVector(x: 0, y: 0, z: 0, w: 1),
        "inputMaxComponents": CIVector(x: 1, y: 1, z: 1, w: 1)])
```

**진단 요령: 색이 이상하면 눈으로 보지 말고 `ctx.render(_:toBitmap:)`으로 min/max/avg를 찍어라.** 추측 세 번보다 한 번의 숫자가 빨랐다.

### 2026-09-02 — 큰 블러는 1/4 해상도에서 (Fadeo 1.1)

할레이션 반경이 짧은 변의 2%라 프리뷰에서도 20px이 넘는데, 가우시안 비용은 반경에 비례한다. 뷰파인더 GPU가 **1.1ms → 8.2ms**로 뛰었다(실기기 iPhone 16, 프레임당).

축소 → 블러 → 되키우면 **결과가 어차피 뭉갠 빛이라 눈으로는 같고 비용은 1/16**이다. 6.6ms까지 내려갔다.

**입자 크기는 픽셀이 아니라 이미지 대비 비율로 잡되 하한을 둔다.** 고정 픽셀이면 렌더 해상도마다 굵기가 달라져 같은 사진이 앨범과 상세에서 다른 필름처럼 보이고, 순수 비례면 앨범 2열 썸네일(약 540px)에서 알갱이가 1.6px까지 작아져 **필름 입자가 아니라 디지털 노이즈로 보인다.**


### 2026-08-25 — 300pt 카드에 그리려고 12MP를 통째로 디코드하고 있었다 (즉석카메라)

현상 진행을 보여주는 뷰가 매번 ① 원본 JPEG 전체 디코드 → ② `CIImage` → ③ 필터 체인 → ④ `createCGImage` → ⑤ 풀해상도 `UIImage`를 `@State`에 보관까지 **메인 스레드에서** 했다. 12MP 기준 정사각 크롭 후 3024×3024, RGBA로 **장당 약 36MB가 뷰마다 상주**한다. 2열 그리드에서 셀 여섯이면 200MB대 — jetsam 사정권이고, 현상 중인 사진은 5초마다 그 비용을 다시 치렀다.

**고칠 곳은 필터가 아니라 디코드다.** `CGImageSourceCreateThumbnailAtIndex`는 전체 디코드를 **건너뛰고** 목표 크기로 바로 뽑는다.

```swift
let src = CGImageSourceCreateWithURL(url as CFURL,
    [kCGImageSourceShouldCache: false] as CFDictionary)!
let cg = CGImageSourceCreateThumbnailAtIndex(src, 0, [
    kCGImageSourceCreateThumbnailFromImageAlways: true,
    kCGImageSourceShouldCacheImmediately: true,
    kCGImageSourceCreateThumbnailWithTransform: true,     // EXIF 방향 반영
    kCGImageSourceThumbnailMaxPixelSize: Int(maxPixel)
] as CFDictionary)!
```

- 목표는 **pt가 아니라 pt × `UIScreen.main.scale`**이다. 300pt 카드면 900px. 픽셀 수가 (900/3024)² ≈ **1/11**로 준다.
- `kCGImageSourceCreateThumbnailWithTransform: true`가 EXIF 방향까지 반영해준다 — 아래 항목의 함정을 이 경로에서는 공짜로 피한다.
- **`CIContext`는 스레드 안전하다.** 렌더 전체를 `Task.detached`로 옮기고 결과 `UIImage`만 `MainActor`로 되돌리면 된다.

### 2026-08-25 — `CIImage(image:)`는 `UIImage.imageOrientation`을 무시한다 ★ (즉석카메라)

`AVCapturePhotoOutput`의 `fileDataRepresentation()`은 세로 촬영 시 **픽셀은 가로로, 방향은 EXIF orientation(보통 6)으로** 저장한다. `UIImage(data:)`는 그걸 `imageOrientation` 속성으로 읽지만, **`CIImage(image:)`는 그 속성을 버리고 원본 CGImage 픽셀만 가져간다.** 게다가 `UIImage(cgImage:)`는 방향이 `.up` 고정이라, 파이프라인을 통과하고 나면 세로 사진이 90도 돌아간 채로 표시·내보내기된다.

우회 셋 중 **저장 시점에 픽셀을 세워 굽는 것**이 가장 낫다 — 렌더할 때마다 방향 보정을 반복하지 않아도 된다.

```swift
guard let ui = UIImage(data: data), ui.imageOrientation != .up else { return nil }  // 이미 .up이면 재인코딩 안 함
let upright = UIGraphicsImageRenderer(size: ui.size, format: fmt).image { _ in
    ui.draw(in: CGRect(origin: .zero, size: ui.size))
}
```

**시뮬레이터에서는 절대 재현되지 않는다.** 번들에 넣어 테스트하는 JPG는 방향이 `.up`이라 통과한다 — 이 종류의 버그는 실기기에서만 드러난다.

### 2026-08-25 — 즉석카메라 10분 현상 곡선

- 맥락: [[프로젝트/개인/즉석카메라/README|즉석카메라]] 마일스톤 0. 앱을 만들기 전에 현상 연출만 떼어내 검증했다.
- 배운 것: 위 6개 항목 전부. 특히 ① **색조 이동의 적정 폭(±10% 안쪽)** ② **블랙 리프트는 bias가 아니라 톤커브** ③ **색 판정에는 UI 맥락이 필요하다**.
- 근거: `(로컬 경로)` — 1차(R 0.45배)는 전 구간이 파랗게 나와 폐기, 2차(R −13%)에서 정상화. 상세는 [[프로젝트/개인/즉석카메라/즉석카메라 개발 기록 2026-08-25|개발 기록 2026-08-25]].

## 참고 자료

- [Core Image Filter Reference — CIToneCurve / CITemperatureAndTint / CIColorMatrix](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html) — 필터별 파라미터 정의 (2026-08-25 확인)
