# SDK 시작하기

이 페이지에서는 nap mx Android SDK를 프로젝트에 추가하고 초기화하는 방법을 안내합니다.

---

## 📱 실제 연동 서비스 앱 참고 (바이오리듬)

SDK를 연동하기 전, 구글 플레이 스토어에 출시된 **바이오리듬(Biorhythm) 앱**을 설치하여 SDK를 이용한 실제 광고 소재가 어떻게 송출되고 구동되는지 직접 확인해 보실 수 있습니다.
*(본 앱은 테스트용으로 제공하는 데모 앱이 아니며, 실제 상용 서비스 앱에 SDK가 어떻게 적용되어 화면을 구성하는지 참고할 수 있는 실제 연동 사례입니다.)*

<table style="border: none; background: transparent;">
  <tr style="border: none; background: transparent;">
    <td style="border: none; padding-right: 15px; vertical-align: middle;">
      <img src="https://play-lh.googleusercontent.com/yBai-9NRf3BkcYuSS9jU2MeGRXnpNx-Mn9EEKegP-BvdtFtzeCJDgMFMRLrnHTXwCX4AFh9kNbgVLlPNfrQ82dY" alt="Biorhythm Icon" width="64" height="64" style="border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);"/>
    </td>
    <td style="border: none; vertical-align: middle;">
      <strong>함께하는 바이오리듬 (Biorhythm)</strong><br/>
      <a href="https://play.google.com/store/apps/details?id=kr.co.nasmedia.biorhythm" target="_blank" style="text-decoration: none;">
        <img src="https://upload.wikimedia.org/wikipedia/commons/7/78/Google_Play_Store_badge_EN.svg" alt="Google Play에서 다운로드" width="120" style="margin-top: 4px;"/>
      </a>
    </td>
  </tr>
</table>

* 이 앱에서는 SDK가 제공하는 실제 광고 포맷(Basic 전면, 배너 및 네이티브 등)이 실제 사용자 화면에서 어떻게 송출되는지 레퍼런스로 참고하실 수 있습니다.

---

## Step 1. Gradle 설정

### 1-1. 프로젝트 최상위 `build.gradle`

```gradle
allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

### 1-2. 앱 모듈 `build.gradle`

> ⚠️ 라이브러리 버전은 항상 **최신 버전**으로 유지하세요. 구버전 사용 시 광고 수신율이 저하되거나 보안 취약점이 발생할 수 있습니다.

의존성은 아래 두 방식 중 하나를 선택합니다. **BOM(방법 A)을 권장**합니다 — 멤버 버전을 한 곳에서 고정해 버전 불일치·구버전 사용을 방지합니다.

#### 방법 A — BOM 사용 (권장)

`admixer-bom`을 import하면 각 멤버는 **버전을 생략**해도 BOM이 지정한 버전으로 자동 고정됩니다.

```gradle
dependencies {
    // ✅ 필수 — BOM import (한 줄로 모든 admixer 멤버 버전 고정)
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.07.02')

    // ✅ 필수 — Core SDK (버전 생략 = BOM이 관리)
    implementation 'io.github.nasmedia-tech:admixer-ssp'
    // ✅ 필수 — Google Advertising ID
    implementation 'com.google.android.gms:play-services-ads-identifier:18.2.0'

    // 선택 — 사용하는 미디에이션 네트워크만 추가 (버전 생략)
    implementation 'io.github.nasmedia-tech:admixer-admanager'       // Google AdManager
    implementation 'io.github.nasmedia-tech:admixer-adfit'           // Kakao Adfit
    implementation 'io.github.nasmedia-tech:admixer-pangle'          // Pangle
    implementation 'io.github.nasmedia-tech:admixer-applovin'        // AppLovin
    implementation 'io.github.nasmedia-tech:admixer-unity'           // Unity Ads
    implementation 'io.github.nasmedia-tech:admixer-naveradmanager'  // Naver Ad Manager
    implementation 'io.github.nasmedia-tech:admixer-teads'           // Teads
    implementation 'io.github.nasmedia-tech:admixer-unity-nativeadlayout'  // Unity 네이티브 레이아웃 헬퍼 (선택 — 직접 NativeAdViewBinder 레이아웃을 구성하면 불필요)

    // 🧪 (beta) — Google Mobile Ads NextGen SDK. admixer-admanager와 택1 (아래 주의 참고)
    // implementation 'io.github.nasmedia-tech:admixer-gma-nextgen'
}
```

#### 방법 B — 개별 버전 지정

BOM 없이 각 아티팩트 버전을 직접 명시합니다. (아래는 **현재 배포된 최신 버전** 기준 — 이후 [Maven Central](https://central.sonatype.com/namespace/io.github.nasmedia-tech)에서 최신 버전을 확인해 갱신하세요.)

```gradle
dependencies {
    // ✅ 필수 — Core SDK
    implementation 'io.github.nasmedia-tech:admixer-ssp:2.1.0'
    // ✅ 필수 — Google Advertising ID
    implementation 'com.google.android.gms:play-services-ads-identifier:18.2.0'

    // 선택 — 사용하는 미디에이션 네트워크만 추가하세요
    implementation 'io.github.nasmedia-tech:admixer-admanager:2.0.1'       // Google AdManager (play-services-ads:25.2.0 포함)
    implementation 'io.github.nasmedia-tech:admixer-adfit:2.0.2'           // Kakao Adfit (ads-base:3.21.17 포함)
    implementation 'io.github.nasmedia-tech:admixer-pangle:2.0.1'          // Pangle (pag-sdk:8.0.0.5 포함)
    implementation 'io.github.nasmedia-tech:admixer-applovin:2.0.1'        // AppLovin (applovin-sdk:13.6.3 포함)
    implementation 'io.github.nasmedia-tech:admixer-unity:2.0.1'           // Unity Ads (unity-ads:4.18.1 포함)
    implementation 'io.github.nasmedia-tech:admixer-naveradmanager:2.0.1'  // Naver Ad Manager (nam-bom:8.16.0 포함)
    implementation 'io.github.nasmedia-tech:admixer-teads:2.0.1'           // Teads (teads-sdk:6.1.0 포함)
    implementation 'io.github.nasmedia-tech:admixer-unity-nativeadlayout:2.0.0'  // Unity 네이티브 레이아웃 헬퍼 (선택 — admixer-unity와 함께, 직접 NativeAdViewBinder 레이아웃 구성 시 불필요)

    // 🧪 (beta) — Google Mobile Ads NextGen SDK (ads-mobile-sdk:1.2.1 포함). admixer-admanager와 택1
    // implementation 'io.github.nasmedia-tech:admixer-gma-nextgen:2.0.1'
}
```

---

### 🧪 GMA NextGen (beta) — 도입 전 필독

Google이 차세대로 발표한 **Mobile Ads NextGen SDK** 연동 어댑터입니다. **beta 단계**이며, 기존 `admixer-admanager`(classic)를 대체하는 별도 모듈입니다.

> 🚨 **classic `play-services-ads`를 쓰는 어댑터와 같은 앱에 넣을 수 없습니다.**
> NextGen SDK는 classic `com.google.android.gms:play-services-ads`의 **전역 exclude**를 요구합니다. 따라서:
>
> | 어댑터 | NextGen과 공존 |
> |---|---|
> | `admixer-admanager` (classic) | ❌ **불가** — 택1 |
> | `admixer-naveradmanager` | ❌ **불가** — 내부적으로 GAM 미디에이션(nam-dfp) 사용 |
> | `admixer-adfit` · `admixer-pangle` · `admixer-applovin` · `admixer-unity` · `admixer-teads` | ✅ 가능 |
>
> **국내 지면은 대부분 AdManager·NaverAd를 함께 사용하므로, NextGen 도입 시 두 네트워크를 포기해야 합니다.** 도입 전 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의해 지면 구성을 검토받으시길 권장합니다.

사용 시 `build.gradle`에 exclude를 추가하세요.

```gradle
configurations.all {
    exclude group: 'com.google.android.gms', module: 'play-services-ads'
}
```

- **minSdk 24** 이상 필요 (classic AdManager는 23)
- 미디에이션은 Ad Manager 또는 no-mediation만 호환

### 1-3. 네트워크별 추가 Maven 저장소

일부 네트워크는 별도 Maven 저장소 추가가 필요합니다. `settings.gradle`의 `dependencyResolutionManagement` 블록에 추가하세요.

```gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        // Adfit (Kakao) 사용 시 필수
        maven { url 'https://devrepo.kakao.com/nexus/content/groups/public/' }
        // Pangle 사용 시 필수
        maven { url "https://artifact.bytedance.com/repository/pangle/" }
        // Teads 사용 시 필수
        maven { url "https://sdk.teads.tv/android/repo" }
        maven { url "https://teads.jfrog.io/artifactory/SDKAndroid-maven-prod" }
    }
}
```

| 네트워크 | 필요 저장소 |
|---------|------------|
| Google AdManager, AppLovin, Unity, NaverAdManager | `google()` / `mavenCentral()` 만으로 해결 |
| Kakao Adfit | `devrepo.kakao.com` 추가 필요 |
| Pangle | `artifact.bytedance.com` 추가 필요 |
| Teads | `sdk.teads.tv`, `teads.jfrog.io` 추가 필요 |

### 1-4. 네트워크 SDK 지원 버전 범위

각 어댑터(`admixer-*`)는 아래 **번들(검증) 버전**으로 빌드·검증되었습니다. 어댑터 aar에는 해당 네트워크 SDK가 전이 의존으로 포함되므로, 별도로 버전을 지정하지 않아도 번들 버전이 자동 적용됩니다.

> ℹ️ **최소 / 최대**는 각 네트워크 벤더의 릴리스 호환성 기준 권장 범위입니다. 어댑터가 호출하는 SDK 클래스·메서드·인자가 해당 범위에서 존재·동일한지 **소스 레벨로 교차검증**하고, 실제 Gradle resolve 결과 및 네이티브 라이브러리 16KB 정렬까지 **전수 재점검**(2026-07-12)했습니다. 다만 운영 적용 전 해당 버전으로 빌드·동작을 한 번 더 확인하는 것을 권장합니다.

| 네트워크 | Maven 라이브러리 | 최소 지원 | 번들(검증) | 최대 호환 | 비고 |
|---|---|---|---|---|---|
| AdMixer (Core) | `io.github.nasmedia-tech:admixer-ssp` | 2.0.0 | **2.1.0** | 2.1.0 | 자체 SDK |
| Google AdManager | `com.google.android.gms:play-services-ads` | 24.0.0 | **25.2.0** | 25.2.0 | ⚠️ **25.3.0+ 비호환**(상한 고정) |
| Kakao Adfit | `com.kakao.adfit:ads-base` | 3.17.2 | **3.21.17** | 3.22.2 | 3.x 단일 라인 |
| Pangle | `com.pangle.global:pag-sdk` | 8.0.0.4 | **8.0.0.5** | 8.1.0.3 | 8.x 라인 권장 |
| AppLovin | `com.applovin:applovin-sdk` | 13.2.0 (권장 13.4.0+) | **13.6.3** | 13.6.3 | 12.x 이하 미지원 |
| Unity Ads | `com.unity3d.ads:unity-ads` | 4.16.x (권장 4.18.0) | **4.18.1** | 4.18.1 | 4.x 라인 |
| Naver Ad Manager | `com.naver.gfpsdk:nam-bom` | 8.14.0 | **8.16.0** | 8.17.0 | 8.x(BOM이 모듈 버전 고정) |
| Teads | `tv.teads.sdk.android:sdk` | 6.0.4 (권장 6.1.0) | **6.1.0** | 6.1.0 | 6.x 통합 SDK(5.x는 레거시) |
| 🧪 GMA NextGen **(beta)** | `com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk` | 1.2.1 | **1.2.1** | 1.2.1 | AdManager·NaverAd와 공존 불가 |

> ⚠️ **Google AdManager (`play-services-ads`)는 25.2.0 상한을 반드시 지키세요.** 25.3.0+는 호환 이슈가 있어, 다른 어댑터의 전이 의존이 상위 버전을 끌어오지 못하도록 강제 고정을 권장합니다.
> ```gradle
> configurations.all {
>     resolutionStrategy {
>         force 'com.google.android.gms:play-services-ads:25.2.0'
>     }
> }
> ```

#### 네트워크별 최소 Android API (런타임 요구)

번들된 네트워크 SDK가 요구하는 최소 Android API가 코어 SDK(API 21)보다 높을 수 있습니다. 해당 어댑터를 추가하면 **호스트 앱의 `minSdkVersion`이 아래 값 이상**이어야 합니다.

| 네트워크 | 최소 Android API |
|---|---|
| AdMixer (Core), Kakao Adfit, Teads | API 21 (Android 5.0) |
| Google AdManager, Pangle, Unity Ads, Naver Ad Manager | API 23 (Android 6.0) |
| AppLovin | API 24 (Android 7.0) — `applovin-sdk:13.6.3` 기준 |
| 🧪 GMA NextGen **(beta)** | API 24 (Android 7.0) — NextGen SDK 요구 |

### 1-5. Kotlin 툴체인 요구사항

일부 네트워크 SDK는 최신 Kotlin 런타임(`kotlin-stdlib`)을 전이 의존으로 포함합니다. 아래 어댑터를 추가하는 경우 **호스트 앱을 Kotlin 2.0 이상(권장 2.1+)으로 빌드**하세요. 구 Kotlin 컴파일러로 빌드 시 `Module was compiled with an incompatible version of Kotlin` 경고/오류가 발생할 수 있습니다.

| 사용 구성 | 매체 앱 최소 Kotlin | 근거 |
|---|---|---|
| **코어(`admixer-ssp`)만** | Kotlin 1.8+ / **Java-only 가능** | 코어는 낮은 stdlib만 요구 |
| + Google AdManager | **Kotlin 2.1+** | GMA가 최소 Kotlin 2.1.0 요구 |
| + Kakao Adfit | **Kotlin 2.0+** | ads-base가 stdlib 2.0.x + coroutines 유입 |
| + Naver Ad Manager | **Kotlin 2.1+** | GFP SDK(Kotlin 기반) |
| + 🧪 GMA NextGen **(beta)** | **Kotlin 2.1+** | GMA 계열과 동일 |
| + Pangle / AppLovin / Unity / Teads | 영향 없음 | 모두 Java 기반, Kotlin 미강제 |

> ℹ️ Java 8/11/17 모두 무방합니다. 코어만 임베드하는 앱의 지원 범위(Kotlin 1.8+/Java-only)는 변하지 않습니다. 위 상승은 각 네트워크 SDK 벤더가 요구하는 최소치이며 SDK가 추가로 강제한 것이 아닙니다.

### 1-6. Android 15 · 16KB 페이지 크기

Google Play는 Android 15 이상 타깃 앱에 대해 64비트 네이티브 라이브러리의 **16KB 페이지 정렬**을 요구합니다. 네이티브 라이브러리를 포함하는 **Pangle·AppLovin**의 `.so`가 모두 16KB 정렬로 출하됨을 실측 확인했으며(그 외 네트워크는 네이티브 라이브러리 미포함), **전 네트워크가 16KB 요건을 충족**합니다. 매체 측 추가 조치는 필요하지 않습니다.

---

## Step 2. AndroidManifest.xml 설정

특정 네트워크를 사용할 경우 `AndroidManifest.xml`에 추가 설정이 필요합니다.

### Google AdManager 사용 시 (필수)

```xml
<application>
    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="nap mx 운영팀으로부터 발급받은 Google App ID" />
</application>
```

### Naver Ad Manager 사용 시

별도 설정이 필요 없습니다. Naver Ad Manager의 `com.naver.gfpsdk.PUBLISHER_CD`는 nap mx가 SDK(`admixer-naveradmanager` aar)에서 제공·관리하므로 **호스트 앱 매니페스트에 설정하지 마세요.** (SDK 동기화 시 자동 포함)

> ℹ️ Google App ID는 **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)**로 문의하여 발급받으세요. (Naver `PUBLISHER_CD`는 SDK가 제공하므로 매체 설정 불필요)

### 네트워크 보안 설정(networkSecurityConfig)을 직접 선언하는 경우

코어 SDK는 광고 네트워크(매체사/DSP) 호환을 위해 `<application>`에 `android:networkSecurityConfig`를 선언합니다. **매체 앱이 자체 `networkSecurityConfig`를 선언하면** 매니페스트 병합 충돌이 발생하므로, 앱 매니페스트 `<application>`에 `tools:replace`를 지정해 매체 설정이 우선하도록 하세요. (매체가 자체 NSC를 선언하지 않으면 별도 조치 불필요)

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <application
        android:networkSecurityConfig="@xml/your_network_security_config"
        tools:replace="android:networkSecurityConfig">
        ...
    </application>
</manifest>
```

---

## Step 3. SDK 초기화

> 🚨 SDK 초기화는 광고 호출 전 앱에서 **반드시 1회** 호출해야 합니다. `Application.onCreate()`에서 호출하는 것을 권장합니다.

#### Java
```java
// MyApplication.java
public class MyApplication extends android.app.Application {

    // 파트너 사이트에서 발급받은 키값으로 교체하세요
    public static final String MEDIA_KEY = "발급받은 미디어 키 **앱당 1개의 미디어키만 적용 가능**";
    public static final String ADUNIT_ID_BANNER = "배너 애드유닛 ID";
    public static final String ADUNIT_ID_INTERSTITIAL = "전면 배너 애드유닛 ID";
    public static final String ADUNIT_ID_NATIVE = "네이티브 애드유닛 ID";
    public static final String ADUNIT_ID_REWARD_VIDEO = "리워드 동영상 애드유닛 ID";
    public static final String ADUNIT_ID_VIDEO = "인라인 동영상 애드유닛 ID";

    @Override
    public void onCreate() {
        super.onCreate();

        // 1. 로그 레벨 설정 (개발 중 VERBOSE, 배포 시 ERROR 권장)
        AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE);

        // 2. 사용할 모든 adunit id를 목록으로 등록
        ArrayList<String> adUnits = new ArrayList<>(Arrays.asList(
            ADUNIT_ID_BANNER,
            ADUNIT_ID_INTERSTITIAL,
            ADUNIT_ID_NATIVE,
            ADUNIT_ID_REWARD_VIDEO,
            ADUNIT_ID_VIDEO
        ));

        // 3. SDK 초기화 — build.gradle에 추가된 어댑터는 자동으로 등록됩니다
        AdMixer.getInstance().initialize(this, MEDIA_KEY, adUnits);

        // ※ Pangle 등 네트워크 SDK는 워터폴에서 어댑터가 자동(lazy) 초기화하므로
        //   Application에서 PAGSdk.init() 등 별도 초기화가 필요하지 않습니다.
        //   Pangle app_id는 media-conf 서버 또는 AdInfo.setAdapterConfig로 전달됩니다.
    }
}
```

#### Kotlin
```kotlin
// MyApplication.kt
class MyApplication : Application() {

    companion object {
        const val MEDIA_KEY = "발급받은 미디어 키"
        const val ADUNIT_ID_BANNER = "배너 애드유닛 ID"
        const val ADUNIT_ID_INTERSTITIAL = "전면 배너 애드유닛 ID"
        const val ADUNIT_ID_NATIVE = "네이티브 애드유닛 ID"
        const val ADUNIT_ID_REWARD_VIDEO = "리워드 동영상 애드유닛 ID"
        const val ADUNIT_ID_VIDEO = "인라인 동영상 애드유닛 ID"
    }

    override fun onCreate() {
        super.onCreate()

        // 1. 로그 레벨 설정
        AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE)

        // 2. 사용할 adunit id 목록
        val adUnits = arrayListOf(
            ADUNIT_ID_BANNER,
            ADUNIT_ID_INTERSTITIAL,
            ADUNIT_ID_NATIVE,
            ADUNIT_ID_REWARD_VIDEO,
            ADUNIT_ID_VIDEO
        )

        // 3. SDK 초기화 — build.gradle에 추가된 어댑터는 자동으로 등록됩니다
        AdMixer.getInstance().initialize(this, MEDIA_KEY, adUnits)

        // ※ Pangle 등 네트워크 SDK는 워터폴에서 어댑터가 자동(lazy) 초기화하므로
        //   Application에서 PAGSdk.init() 등 별도 초기화가 필요하지 않습니다.
        //   Pangle app_id는 media-conf 서버 또는 AdInfo.setAdapterConfig로 전달됩니다.
    }
}
```

### 선택 초기화 옵션 (개인정보 동의 / 테스트)

```java
// 아동 대상 앱 여부 — 아동 대상이라면 필수 (Google Play Families 정책)
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE);  // 아동 대상
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_FALSE); // 일반 대상

// 테스트 모드 / 테스트 디바이스 (QA·심사용)
AdMixer.setTestMode(true);
AdMixer.setTestDeviceIds(Arrays.asList("AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE"));
```

> ℹ️ 네트워크별 전파 매핑은 [개인정보 / 테스트 설정](privacy.md)을 참고하세요.

---

## Step 4. ProGuard 설정

릴리즈 빌드에서 ProGuard/R8 난독화를 사용하는 경우 아래 규칙을 `proguard-rules.pro`에 추가하세요.

```proguard
# ✅ 필수 — AdMixer Core
-keep class com.nasmedia.admixerssp.** { *; }

# 사용하는 어댑터 모듈만 추가하세요
-keep class com.nasmedia.admanager.** { *; }       # Google AdManager
-keep class com.nasmedia.adfit.** { *; }            # Kakao Adfit
-keep class com.nasmedia.pangle.** { *; }           # Pangle
-keep class com.nasmedia.applovin.** { *; }         # AppLovin
-keep class com.nasmedia.unity.** { *; }            # Unity Ads
-keep class com.nasmedia.naveradmanager.** { *; }   # Naver Ad Manager
-keep class com.nasmedia.teads.** { *; }            # Teads
```

> ℹ️ 각 네트워크 SDK가 자체 ProGuard 규칙을 `consumerProguardFiles`로 제공하는 경우, 해당 규칙이 자동으로 적용됩니다. 빌드 경고가 발생하면 해당 네트워크의 ProGuard 가이드를 참고하세요.

---

## 네트워크 SDK 중복 예외 처리

이미 자체/타사 솔루션으로 동일한 네트워크 SDK를 운영 중인 경우, `exclude`로 중복을 방지하세요.

```gradle
dependencies {
    // 이미 Google AdManager SDK를 직접 사용 중인 경우
    implementation("io.github.nasmedia-tech:admixer-admanager:2.0.1") {
        exclude group: "com.google.android.gms", module: "play-services-ads"
    }

    // 이미 Kakao Adfit SDK를 직접 사용 중인 경우
    implementation("io.github.nasmedia-tech:admixer-adfit:2.0.2") {
        exclude group: "com.kakao.adfit", module: "ads-base"
    }

    // 이미 Pangle SDK를 직접 사용 중인 경우
    implementation("io.github.nasmedia-tech:admixer-pangle:2.0.1") {
        exclude group: "com.pangle.global", module: "pag-sdk"
    }
}
```

> ⚠️ exclude 적용 후 반드시 아래를 확인하세요.
> 1. Gradle 의존성 트리에서 동일 네트워크 SDK가 1개만 포함되어 있는지 확인
> 2. 빌드 정상 여부 확인
> 3. nap mx 광고 및 기존 광고 모두 정상 동작 여부 확인

---

## Google SDK 입찰 광고 소스 설정

Google AdManager를 미디에이션으로 사용하는 경우, 아래 광고 소스 라이브러리를 모두 추가해야 최적 수익화가 가능합니다.

* [Google 공식 가이드 — 네트워크 선택](https://developers.google.com/ad-manager/mobile-ads-sdk/android/choose-networks?hl=ko)

**추가해야 할 광고 소스 (모두 추가 권장):**

| 광고 소스 |
|-----------|
| Pangle |
| AppLovin |
| DT Exchange |
| InMobi |
| Liftoff Monetize |
| Meta Audience Network |
| Moloco |
| Unity Ads |
| Mintegral |

> ⚠️ 프로젝트 수준 `build.gradle`과 앱 수준 `build.gradle` **양쪽에 모두** 추가해야 합니다.
