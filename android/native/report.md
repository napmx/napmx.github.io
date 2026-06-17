# 광고 신고하기

nap mx SDK 2.0.0부터 광고 소재에 신고하기 기능이 추가되었습니다.

---

## 신고 아이콘 표시

`AdInfo.Builder`에서 `showReportIcon(true)`를 설정하면 광고 소재 위에 신고 아이콘(ⓘ)이 표시됩니다.

```java
AdInfo adInfo = new AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
    .showReportIcon(true)  // 신고 아이콘 활성화 (기본값: false)
    .build();
```

```kotlin
val adInfo = AdInfo.Builder(MyApplication.ADUNIT_ID_BANNER)
    .showReportIcon(true)
    .build()
```

---

## 동작 방식

1. 광고 소재가 화면에 표시되면 좌측 중앙에 반투명 원형 **ⓘ** 아이콘이 나타납니다.
2. 사용자가 아이콘을 탭하면 광고 신고 화면이 표시됩니다.
3. 신고 완료 시 내부적으로 운영팀에 신고 정보가 전달됩니다.

> ℹ️ 신고 기능은 PixelCopy를 활용한 광고 소재 자동 캡처를 지원합니다 (Android 8.0 이상). 캡처된 소재는 신고 정보와 함께 전달되어 빠른 처리를 돕습니다.

---

## 지원 포맷

| 포맷 | 지원 여부 |
|------|----------|
| 배너 (`AMMBannerView`) | ✅ |
| 전면 배너 (`AMMInterstitial`) | ✅ |
| 네이티브 (`AMMNativeAdView`) | ✅ |
| 동영상 (`AMMVideoView`) | ✅ |
