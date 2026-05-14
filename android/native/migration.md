# v2.0.0 마이그레이션 가이드

이 문서는 nap mx Android SDK **v1.x → v2.0.0** 업그레이드 시 필요한 변경 사항을 설명합니다.

---

## 빠른 요약

| 구분 | 내용 |
|------|------|
| 하위 호환 | 기존 Public API 변경 없음 — 대부분 기존 코드 그대로 동작 |
| 새 기능 | NaverAdManager·Teads 어댑터, 광고 신고 기능 추가 |
| **어댑터 등록 간소화** | **`registerAdapter()` 호출 불필요 — Gradle 의존성 추가만으로 자동 등록** |
| Deprecated | `loadInterstitial()`, `closeInterstitial()`, `onDestroy()` 등 |
| ProGuard | 규칙 강화 — 아래 최신 규칙으로 교체 필요 |
| Gradle 버전 | `2.0.0.SNAPSHOT` → `2.0.0` |

---

## Step 1. Gradle 버전 업데이트

```gradle
// Before
implementation 'io.github.nasmedia-tech:admixer-ssp:1.0.21'
implementation 'io.github.nasmedia-tech:admixer-admanager:1.0.21'
// ...

// After
implementation 'io.github.nasmedia-tech:admixer-ssp:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-admanager:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-adfit:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-pangle:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-applovin:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-unity:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-mobwith:2.0.0'
// 신규 — v2.0.0에서 추가됨
implementation 'io.github.nasmedia-tech:admixer-naveradmanager:2.0.0'
implementation 'io.github.nasmedia-tech:admixer-teads:2.0.0'
```

---

## Step 2. ProGuard 규칙 업데이트

```proguard
# ✅ 필수 — AdMixer Core
-keep class com.nasmedia.admixerssp.** { *; }

# 사용하는 어댑터 모듈만 추가
-keep class com.nasmedia.admanager.** { *; }
-keep class com.nasmedia.adfit.** { *; }
-keep class com.nasmedia.pangle.** { *; }
-keep class com.nasmedia.applovin.** { *; }
-keep class com.nasmedia.unity.** { *; }
-keep class com.nasmedia.mobwith.** { *; }
-keep class com.nasmedia.naveradmanager.** { *; }   # 신규
-keep class com.nasmedia.teads.** { *; }            # 신규
```

> **📌 참고** 각 어댑터 AAR에 `consumer-rules.pro`가 포함되어 있어 대부분의 규칙이 자동 적용됩니다. 위 규칙은 추가 안전망입니다.

---

## Step 3. `registerAdapter()` 호출 제거

v2.0.0부터 `initialize()` 내부에서 클래스패스에 포함된 어댑터를 **자동으로 탐지·등록**합니다.

```java
// Before (v1.x) — 수동 등록 필요
AdMixer.registerAdapter(AdMixer.ADAPTER_ADMANAGER);
AdMixer.registerAdapter(AdMixer.ADAPTER_ADFIT);
// ...

// After (v2.0.0) — 불필요, 제거하세요
```

---

## Step 4. 새 어댑터 추가 (선택)

### NaverAdManager 추가 시

`AndroidManifest.xml`에 Publisher ID를 추가합니다:

```xml
<application>
    <meta-data
        android:name="com.naver.gfpsdk.PUBLISHER_ID"
        android:value="nap mx 운영팀으로부터 발급받은 Publisher ID" />
</application>
```

### Teads 추가 시

`settings.gradle` Maven 저장소 추가:

```gradle
repositories {
    maven { url "https://sdk.teads.tv/android/repo" }
    maven { url "https://teads.jfrog.io/artifactory/SDKAndroid-maven-prod" }
}
```

---

## Step 5. Deprecated API 교체 (권장)

### InterstitialAd

| 기존 (v1.x) | v2.0.0 대체 |
|------------|------------|
| `loadInterstitial()` | `loadAd()` |
| `closeInterstitial()` | `stopInterstitial()` |
| `onDestroy()` | `stopInterstitial()` |

### RewardInterstitialVideoAd

| 기존 (v1.x) | v2.0.0 대체 |
|------------|------------|
| `onDestroy()` | `stopRewardVideoAd()` |

### InterstitialVideoAd

| 기존 (v1.x) | v2.0.0 대체 |
|------------|------------|
| `onDestroy()` | `stopInterstitialVideoAd()` |

### AdInfo.Builder

| 기존 (v1.x) | v2.0.0 대체 |
|------------|------------|
| `.isUseBackgroundAlpha(Boolean)` | `.setUseBackgroundAlpha(boolean)` |

---

## Step 6. 새 기능 적용 (선택)

### 광고 신고하기

```java
AdInfo adInfo = new AdInfo.Builder(ADUNIT_ID)
    .showReportIcon(true)
    .build();
```

---

## 변경 없는 항목

| 클래스 | 변경 여부 |
|--------|----------|
| `AdView` | 변경 없음 |
| `AdListener` | 변경 없음 |
| `AdEvent` | 변경 없음 |
| `NativeAdView` | 변경 없음 |
| `VideoAdView` | 변경 없음 |
| `InterstitialAd` (deprecated 제외) | 변경 없음 |
| `RewardInterstitialVideoAd` (deprecated 제외) | 변경 없음 |
| `InterstitialVideoAd` (deprecated 제외) | 변경 없음 |

---

## 문의

업그레이드 관련 문의사항은 **nap_mx@nasmedia.co.kr**로 연락하세요.
