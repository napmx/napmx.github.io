# SDK 시작하기

이 페이지에서는 nap ssp Android SDK를 프로젝트에 추가하고 초기화하는 방법을 안내합니다.

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

```gradle
dependencies {
    // ✅ 필수 — Core SDK
    implementation 'io.github.nasmedia-tech:admixer-ssp:2.0.0'
    // ✅ 필수 — Google Advertising ID
    implementation 'com.google.android.gms:play-services-ads-identifier:18.2.0'

    // 선택 — 사용하는 미디에이션 네트워크만 추가하세요
    implementation 'io.github.nasmedia-tech:admixer-admanager:2.0.0'       // Google AdManager (play-services-ads:25.2.0 포함)
    implementation 'io.github.nasmedia-tech:admixer-adfit:2.0.0'           // Kakao Adfit (ads-base:3.21.17 포함)
    implementation 'io.github.nasmedia-tech:admixer-pangle:2.0.0'          // Pangle (pag-sdk:8.0.0.5 포함)
    implementation 'io.github.nasmedia-tech:admixer-applovin:2.0.0'        // AppLovin (applovin-sdk:13.6.3 포함)
    implementation 'io.github.nasmedia-tech:admixer-unity:2.0.0'           // Unity Ads (unity-ads:4.18.1 포함)
    implementation 'io.github.nasmedia-tech:admixer-mobwith:2.0.0'         // Mobwith (mobwithSDK:1.0.83 포함, JitPack 저장소 필요)
    implementation 'io.github.nasmedia-tech:admixer-naveradmanager:2.0.0'  // Naver Ad Manager (nam-bom:8.16.0 포함)
    implementation 'io.github.nasmedia-tech:admixer-teads:2.0.0'           // Teads (teads-sdk:6.1.0 포함)

    // 선택 — Unity 네이티브 광고용 기본 레이아웃 헬퍼 (직접 NativeAdViewBinder 레이아웃을 구성하면 불필요)
    implementation 'io.github.nasmedia-tech:admixer-unity-nativeadlayout:2.0.0'  // Unity 네이티브 레이아웃 헬퍼 (admixer-unity와 함께 사용)
}
```

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
        // Mobwith 사용 시 필수 (mobwithSDK 1.0.83+ 전이 의존 com.github.Dimezis:BlurView)
        maven { url 'https://jitpack.io' }
    }
}
```

| 네트워크 | 필요 저장소 |
|---------|------------|
| Google AdManager, AppLovin, Unity, NaverAdManager | `google()` / `mavenCentral()` 만으로 해결 |
| Kakao Adfit | `devrepo.kakao.com` 추가 필요 |
| Pangle | `artifact.bytedance.com` 추가 필요 |
| Teads | `sdk.teads.tv`, `teads.jfrog.io` 추가 필요 |
| Mobwith | `jitpack.io` 추가 필요 (1.0.83+ BlurView 전이 의존) |

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

별도 설정이 필요 없습니다. Naver Ad Manager의 `com.naver.gfpsdk.PUBLISHER_CD`는 nap ssp가 SDK(`admixer-naveradmanager` aar)에서 제공·관리하므로 **호스트 앱 매니페스트에 설정하지 마세요.** (SDK 동기화 시 자동 포함)

> ℹ️ Google App ID는 **nap_mx@nasmedia.co.kr**로 문의하여 발급받으세요. (Naver `PUBLISHER_CD`는 SDK가 제공하므로 매체 설정 불필요)

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
// COPPA(아동 대상 앱) 여부 설정
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE);  // 아동 대상
AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_FALSE); // 일반 대상

// 개인정보 동의 — 워터폴에서 각 네트워크로 자동 전파
AdMixer.setGdprConsent(true);          // GDPR 사용자 동의 (EU)
AdMixer.setCcpaDoNotSell(false);       // CCPA Do-Not-Sell (US)

// 테스트 모드 / 테스트 디바이스 (QA·심사용)
AdMixer.setTestMode(true);
AdMixer.setTestDeviceIds(Arrays.asList("AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE"));
```

> ℹ️ 개인정보 동의·테스트 설정의 네트워크별 전파 매핑은 [개인정보 동의 및 테스트 설정](privacy.md)을 참고하세요.

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
-keep class com.nasmedia.mobwith.** { *; }          # Mobwith
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
    implementation("io.github.nasmedia-tech:admixer-admanager:2.0.0") {
        exclude group: "com.google.android.gms", module: "play-services-ads"
    }

    // 이미 Kakao Adfit SDK를 직접 사용 중인 경우
    implementation("io.github.nasmedia-tech:admixer-adfit:2.0.0") {
        exclude group: "com.kakao.adfit", module: "ads-base"
    }

    // 이미 Pangle SDK를 직접 사용 중인 경우
    implementation("io.github.nasmedia-tech:admixer-pangle:2.0.0") {
        exclude group: "com.pangle.global", module: "pag-sdk"
    }
}
```

> ⚠️ exclude 적용 후 반드시 아래를 확인하세요.
> 1. Gradle 의존성 트리에서 동일 네트워크 SDK가 1개만 포함되어 있는지 확인
> 2. 빌드 정상 여부 확인
> 3. nap ssp 광고 및 기존 광고 모두 정상 동작 여부 확인

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
