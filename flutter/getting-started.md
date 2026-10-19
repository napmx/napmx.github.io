# Flutter 설치와 초기화

이 페이지는 빈 Flutter 프로젝트에 플러그인을 설치하고 nap mx 초기화까지 완료하는 순서입니다. Android와 iOS의 발급값 형식과 네트워크 초기화 방식이 다르므로 두 플랫폼 설정을 각각 확인하세요.

## 1. 준비할 값

nap mx 운영 담당자를 통해 다음 값을 발급받습니다.

- 앱별 Media Key
- 사용할 광고 포맷별 AdUnit ID
- 실제 사용하는 미디에이션 네트워크의 App ID 또는 Publisher Key
- 운영 지면과 분리된 테스트 지면

공개 범용 테스트 ID는 제공되지 않습니다. 테스트 ID가 없으면 광고 요청을 보내지 않도록 구현하세요.

| 플랫폼 | Media Key | AdUnit ID |
|---|---|---|
| Android | 문자열 | 문자열 |
| iOS | 숫자로 변환 가능한 문자열 | 숫자로 변환 가능한 문자열 |

Dart API는 두 플랫폼을 공통 지원하므로 모두 `String`을 받습니다. iOS에서 `'12345'`는 허용되지만 `'reward-home'` 같은 값은 `invalid_configuration` 또는 `invalid_ad_unit`으로 거부됩니다.

## 2. 플러그인 설치

현재 `pub.dev`에는 배포하지 않습니다. 재현 가능한 빌드를 위해 `v0.1.6` 태그를 고정합니다. AGP 8 호환 수정은 `v0.1.3`부터 포함됩니다.

> `v0.1.6`은 Android Core 2.4.0 / BOM 2026.10.04, iOS Mediation 2.5.1 / Core 1.3.0을 설치합니다. iOS의 정확한 의존성은 플러그인에 고정되어 있으므로 호스트 Podfile의 버전만 바꿔서 업데이트할 수 없습니다.

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  nap_mx_flutter:
    git:
      url: https://github.com/Nasmedia-Tech/nap_mx_flutter.git
      ref: v0.1.6
```

```bash
flutter pub get
```

## 3. 발급값을 로컬에서 주입

Media Key와 AdUnit ID를 공개 Dart 소스나 CI 로그에 직접 쓰지 마세요. 아래처럼 Git에서 제외한 JSON을 `--dart-define-from-file`로 주입할 수 있습니다.

```json
{
  "NAP_MX_MEDIA_KEY": "발급받은_MEDIA_KEY",
  "NAP_MX_BANNER_ID": "발급받은_BANNER_ID",
  "NAP_MX_REWARDED_ID": "발급받은_REWARDED_ID"
}
```

```gitignore
# 로컬/CI에서 별도로 주입하는 실제 광고 설정
config/local.json
```

```bash
flutter run --dart-define-from-file=config/local.json
```

```dart
// 컴파일 시 주입한 값입니다. 값 자체는 로그에 출력하지 않습니다.
const mediaKey = String.fromEnvironment('NAP_MX_MEDIA_KEY');
const bannerId = String.fromEnvironment('NAP_MX_BANNER_ID');
const rewardedId = String.fromEnvironment('NAP_MX_REWARDED_ID');
```

`--dart-define`은 공개 저장소 커밋을 막는 방법이지 서버 비밀 저장소가 아닙니다. 앱 바이너리에 포함되는 값은 추출될 수 있으므로 관리 API 토큰, 서명키 등은 넣지 마세요.

## 4. Android 설정

`v0.1.3`의 기준은 **AGP 8.10.1 이상, Kotlin 2.1.0 이상(권장 2.2.10), compile SDK 36, Java 17**입니다. 플러그인 자체는 AGP·Kotlin 버전을 고정하지 않으며 호스트의 설정을 사용합니다. 예제의 AGP 8.10.1 조합은 Gradle 8.11.1을 사용합니다. 다른 AGP 버전은 해당 AGP가 요구하는 Gradle 버전을 맞추세요.

AGP 8 호스트의 `android/settings.gradle.kts`에서 Kotlin 플러그인 버전을 선언합니다. Flutter 로더와 기존 AGP 선언은 유지하세요.

```kotlin
plugins {
    // 기존 Flutter 로더·AGP 선언과 함께 추가합니다.
    id("org.jetbrains.kotlin.android") version "2.2.10" apply false
}
```

앱 모듈에서는 Android 플러그인 다음, Flutter 플러그인 전에 Kotlin 플러그인을 적용합니다.

```kotlin
// android/app/build.gradle.kts
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("dev.flutter.flutter-gradle-plugin")
}
```

플러그인 모듈에는 호스트의 Kotlin 플러그인이 자동 적용되므로 로컬 복사본을 수정할 필요가 없습니다. `admixer-ssp:2.4.0`의 stdlib 의존성은 2.2.10이며 Kotlin 컴파일러 2.1.0에서도 빌드를 검증했습니다. stdlib를 컴파일러 버전에 맞춰 강제로 낮추지 마세요.

Core 2.4.0도 stdlib 2.2.10을 사용합니다. 선택 어댑터의 버전 묶음은 아래 최신 BOM으로 정렬합니다. 앱에서 최종 해석 버전과 빌드를 확인하세요.

AGP 9에서 내장 Kotlin을 끈 호스트(`android.builtInKotlin=false`)도 위 Kotlin 플러그인 설정이 필요합니다. Flutter 3.44에서는 `android.newDsl=false`도 유지하세요. 내장 Kotlin을 켠 AGP 9 호스트에는 별도 Kotlin Android 플러그인을 적용하지 않습니다. Flutter 앱의 내장 Kotlin 연동은 Flutter 3.47 이상이 필요합니다. [Flutter 공식 가이드](https://docs.flutter.dev/release/breaking-changes/migrate-to-built-in-kotlin/for-plugin-authors)를 참고하세요.

플러그인이 nap mx Core를 가져옵니다. 매체 앱은 사용하는 네트워크 어댑터만 추가합니다.

```kotlin
// android/app/build.gradle.kts
dependencies {
    // nap mx 모듈을 공식 검증 조합으로 정렬합니다.
    implementation(platform("io.github.nasmedia-tech:admixer-bom:2026.10.04"))

    // 실제 사용하는 네트워크만 주석을 해제합니다.
    implementation("io.github.nasmedia-tech:admixer-admanager")
    // implementation("io.github.nasmedia-tech:admixer-naveradmanager")
    // implementation("io.github.nasmedia-tech:admixer-adfit")
    // implementation("io.github.nasmedia-tech:admixer-pangle")
    // implementation("io.github.nasmedia-tech:admixer-applovin")
    // implementation("io.github.nasmedia-tech:admixer-unity")
    // implementation("io.github.nasmedia-tech:admixer-teads")

    // Beta: classic admixer-admanager와 동시에 추가하지 않습니다.
    // implementation("io.github.nasmedia-tech:admixer-gma-nextgen")
}
```

GMA NextGen은 Android API 24 이상이 필요하며 classic Google Mobile Ads와 함께 사용할 수 없습니다. 선택했다면 [Android 시작 가이드](/android/native/getting-started)의 exclude 설정도 적용하세요.

### Google 광고 ID 버전 확인

Core 2.4.0은 API 21 호환 `play-services-ads-identifier:18.0.0`을 포함합니다. Google 최신 수정판 `18.3.0`은 API 23 이상이 필요합니다. `18.2.0`은 구형 Android에서 `java.time.Duration` 크래시가 있어 사용하지 마세요([Google 릴리스 노트](https://developers.google.com/android/guides/releases#january_27_2026)).

Flutter `v0.1.6`부터는 `18.2.0` 직접 선언이 없어 Core의 `18.0.0`이 적용됩니다. `minSdk` 23 이상 앱은 수정판 `18.3.0`을 다음처럼 추가할 수 있습니다. 최신 AdManager/Naver/GMA NextGen 어댑터를 사용하면 `18.3.0`이 전이 의존성으로 적용됩니다.

```kotlin
// android/app/build.gradle.kts — minSdk 23 이상
dependencies {
    implementation("com.google.android.gms:play-services-ads-identifier:18.3.0")
}
```

```bash
cd android
./gradlew :app:dependencyInsight --dependency play-services-ads-identifier --configuration releaseRuntimeClasspath
```

### Google AdManager 버전 상한

`admixer-admanager`를 사용한다면 `play-services-ads`를 25.2.0으로 고정하세요. 25.3.0 이상은 호환되지 않습니다. Flutter 앱은 다른 광고·분석 플러그인이 같은 아티팩트를 전이 의존으로 함께 끌어오는 경우가 많고, Gradle은 그중 가장 높은 버전을 선택하므로 고정하지 않으면 상한을 넘긴 버전으로 해석될 수 있습니다.

```kotlin
// android/app/build.gradle.kts — admixer-admanager를 사용할 때만 적용합니다.
configurations.configureEach {
    resolutionStrategy {
        force("com.google.android.gms:play-services-ads:25.2.0")
    }
}
```

실제로 어떤 버전이 선택됐는지는 아래로 확인합니다.

```bash
cd android
# 뒤에 콜론을 붙여야 play-services-ads-identifier 가 함께 걸리지 않습니다.
./gradlew :app:dependencies --configuration releaseRuntimeClasspath | grep "play-services-ads:"
```

AdFit, Pangle, Teads를 사용할 때만 `android/build.gradle.kts`의 `allprojects` 블록에 해당 Maven 저장소를 추가합니다. 기본 Flutter 템플릿은 이 블록에서 저장소를 선언하므로, `settings.gradle.kts`의 `dependencyResolutionManagement`에 추가한 저장소는 사용되지 않습니다.

```kotlin
// android/build.gradle.kts
allprojects {
    repositories {
        google()
        mavenCentral()

        maven("https://devrepo.kakao.com/nexus/content/groups/public/") // AdFit
        maven("https://artifact.bytedance.com/repository/pangle/")      // Pangle
        maven("https://sdk.teads.tv/android/repo")                      // Teads
        maven("https://teads.jfrog.io/artifactory/SDKAndroid-maven-prod")
        maven("https://developer.huawei.com/repo/")
    }
}
```

Google Ad Manager를 사용한다면 발급받은 App ID를 `<application>` 안에 추가합니다.

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<application ...>
    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="발급받은 Google App ID" />
</application>
```

선택 네트워크가 요구하는 권한·Manifest 항목과 R8 규칙은 [Android 네이티브 가이드](/android/native/getting-started)를 함께 적용합니다. 플러그인의 consumer 규칙은 자동 병합됩니다.

### Google SDK 입찰 광고 소스 (Android)

`admixer-admanager`를 사용한다면 Google SDK 입찰 광고 소스 어댑터를 함께 추가해야 최적 수익화가 가능합니다. 대상 광고 소스는 [Android 네이티브 가이드](/android/native/getting-started)의 "Google SDK 입찰 광고 소스 설정"과 같습니다. Dart 코드나 `NapMxConfiguration` 설정은 필요 없고 호스트 앱의 Gradle 의존성만 추가합니다.

위에서 `play-services-ads`를 25.2.0으로 고정했으므로 어댑터도 **25.2.0 이하를 요구하는 버전**으로 명시합니다. 최신 어댑터는 25.3.0 이상을 기대하는데 `force`가 25.2.0으로 낮춰, 빌드는 성공해도 런타임에 입찰이 실패할 수 있습니다.

```kotlin
// android/app/build.gradle.kts — play-services-ads 25.2.0 호환 버전
dependencies {
    implementation("com.google.ads.mediation:pangle:8.0.0.5.0")
    implementation("com.google.ads.mediation:applovin:13.6.2.0")
    implementation("com.google.ads.mediation:fyber:8.4.5.0")      // DT Exchange
    implementation("com.google.ads.mediation:inmobi:11.3.0.0")   // inmobi
    implementation("com.google.ads.mediation:vungle:7.7.4.1")     // Liftoff Monetize
    implementation("com.google.ads.mediation:moloco:4.8.1.0")   // moloco
    implementation("com.unity3d.ads:unity-ads:4.18.0")            // Unity Ads SDK — 어댑터에 포함되지 않아 함께 추가
    implementation("com.google.ads.mediation:unity:4.18.0.0")
    implementation("com.google.ads.mediation:mintegral:17.1.61.0") // mintegral
}
```

Unity Ads 어댑터는 다른 어댑터와 달리 Unity Ads SDK(`unity-ads`)를 의존성으로 끌어오지 않습니다. `admixer-unity`를 쓰지 않는다면 반드시 `unity-ads`를 직접 추가하세요. 빠뜨리면 빌드는 성공하지만 런타임에 Unity 입찰 어댑터가 초기화되지 않습니다.

위 버전은 2026-10-12 Google Maven 기준으로, 각 광고 소스에서 `play-services-ads` 25.2.0 이하를 요구하는 가장 최신 버전입니다. Pangle은 `admixer-pangle`과 같은 `pag-sdk` 8.0.0.5를 사용합니다. AppLovin·Unity Ads는 nap mx 어댑터와 함께 쓰면 SDK가 각각 13.6.3·4.18.1로 해석되므로 실기기 확인 결과를 운영 담당자와 공유하세요.

Pangle과 Mintegral은 별도 Maven 저장소가 필요합니다. Pangle 저장소는 `admixer-pangle`을 쓰지 않아도 입찰 광고 소스만으로 필요합니다. 위 저장소 설정과 같이 `android/build.gradle.kts`의 `allprojects` 블록에 추가합니다.

```kotlin
// android/build.gradle.kts
allprojects {
    repositories {
        google()
        mavenCentral()
        maven("https://artifact.bytedance.com/repository/pangle/")                         // Pangle
        maven("https://dl-maven-android.mintegral.com/repository/mbridge_android_sdk_oversea") // Mintegral
    }
}
```

입찰 어댑터를 위한 별도 초기화 코드는 추가하지 않습니다. 적용 후 위 `dependencies` 명령으로 `play-services-ads`가 25.2.0으로 해석되는지 다시 확인하세요.

## 5. iOS 설정

플러그인은 `AdMixerMediation` 2.5.1과 `AdMixer` 1.3.0을 고정합니다. Flutter 프로젝트가 Swift Package Manager를 사용하면 Xcode Package Dependencies에 필요한 공식 어댑터만 추가합니다. CocoaPods 프로젝트라면 `ios/Podfile`에 필요한 Pod만 추가합니다.

```ruby
# ios/Podfile — 사용하는 네트워크만 추가
pod 'AdMixerMediationGAM'
# pod 'AdMixerMediationNAM'
# pod 'AdMixerMediationPangle'
# pod 'AdMixerMediationAppLovin'
# pod 'AdMixerMediationUnityAds'
```

Google을 사용할 때는 `GADApplicationIdentifier`가 필요합니다. 앱에서 ATT 권한을 실제 요청할 때만 사용자에게 표시할 문구를 추가합니다.

```xml
<!-- ios/Runner/Info.plist -->
<key>GADApplicationIdentifier</key>
<string>발급받은 Google App ID</string>

<!-- ATT를 요청하는 앱에만 추가합니다. -->
<key>NSUserTrackingUsageDescription</key>
<string>맞춤형 광고 제공을 위해 기기 식별자 사용 권한이 필요합니다.</string>
```

SKAdNetwork 목록은 선택 네트워크와 버전에 따라 달라집니다. 정적인 목록을 복사하지 말고 [iOS 시작 가이드](/ios/native/getting-started)의 최신 목록과 앱 `Info.plist`를 대조하세요. 플러그인/Core의 Privacy Manifest가 앱과 선택 SDK의 개인정보 선언을 대신하지 않으므로 Xcode Privacy Report도 확인합니다.

iOS 선택 네트워크는 `AppDelegate` 또는 활성 `SceneDelegate`에서 별도 초기화가 필요할 수 있습니다. 순서는 다음과 같습니다.

1. CMP 또는 저장된 사용자 선택을 읽습니다.
2. 네트워크 SDK가 초기화 시 읽는 개인정보 신호를 먼저 설정합니다.
3. 설치한 네트워크 SDK만 공식 가이드에 따라 초기화합니다.
4. Flutter에서 `NapMx.initialize`를 호출합니다.

플러그인의 `mediation` 맵은 Android 전용이며 iOS 네트워크 초기화를 대신하지 않습니다. iOS에서 비어 있지 않은 맵을 전달하면 `mediation_configuration_unsupported` 오류를 반환합니다.

### Google SDK 입찰 광고 소스 (iOS)

`AdMixerMediationGAM`을 사용한다면 Google SDK 입찰 광고 소스 어댑터를 함께 추가합니다. 대상 광고 소스, SPM Repository URL과 고정 버전은 [iOS 네이티브 가이드](/ios/native/getting-started)의 "1-3. Google 네트워크 - SDK 입찰 광고 소스 설정"과 같습니다. 호스트 앱의 `ios/` 프로젝트에만 추가하며, SPM과 CocoaPods에 같은 어댑터를 중복 연결하지 마세요.

```ruby
# ios/Podfile — Runner 타깃
pod 'AdMixerMediationGAM'

pod 'GoogleMobileAdsMediationPangle'     # nap mx Pangle 사용 시 '7.9.0.6.0'으로 고정
pod 'GoogleMobileAdsMediationAppLovin'   # nap mx AppLovin 사용 시 '13.5.1.0'으로 고정
pod 'GoogleMobileAdsMediationVungle'     # Liftoff Monetize
pod 'GoogleMobileAdsMediationMintegral'  # Mintegral
pod 'GoogleMobileAdsMediationFyber'      # DT Exchange
pod 'GoogleMobileAdsMediationInMobi'     # InMobi
pod 'GoogleMobileAdsMediationMoloco'     # Moloco
pod 'GoogleMobileAdsMediationUnity'      # nap mx Unity Ads 사용 시 추가하지 않음
```

SPM 프로젝트는 `ios/Runner.xcworkspace`에서 **File > Add Package Dependencies**로 iOS 네이티브 가이드 표의 Repository URL을 `Runner` 타깃에 추가합니다.

| 조건 | 입찰 어댑터 | CocoaPods | SPM |
|---|---|---|---|
| nap mx Pangle도 사용 | Pangle | `7.9.0.6.0` | `7.9.600` |
| nap mx AppLovin도 사용 | AppLovin | `13.5.1.0` | `13.5.100` |
| nap mx Unity Ads도 사용 | Unity Ads | 추가하지 않음 | 추가하지 않음 |

nap mx 어댑터가 Pangle `Ads-Global`을 8.2.2 미만, `AppLovinSDK`를 13.5.2 미만으로 제한하므로 최신 입찰 어댑터를 그대로 쓰면 `pod install` 또는 패키지 해석이 실패합니다. 해석이 실패하면 버전을 임의로 올리지 말고 운영 담당자에게 문의하세요.

iOS는 앱이 Google Mobile Ads SDK를 직접 시작해야 입찰 어댑터도 초기화됩니다. 개인정보 신호 설정 뒤, Flutter의 `NapMx.initialize`보다 먼저 호출합니다.

```swift
// ios/Runner/AppDelegate.swift
import GoogleMobileAds

override func application(
  _ application: UIApplication,
  didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
) -> Bool {
  // AMMediation.shared.setConsent(consent)를 사용한다면 이 줄보다 먼저 호출합니다.
  MobileAds.shared.start()
  return super.application(application, didFinishLaunchingWithOptions: launchOptions)
}
```

### Google 입찰 어댑터 작동 확인

입찰 광고 소스를 추가한 뒤에는 Google Mobile Ads SDK의 Debug Options Menu로 입찰 광고 소스와 어댑터 상태를 확인합니다. 플러그인은 이 메뉴를 여는 Dart API를 제공하지 않으므로, 테스트 빌드에서 호스트 앱이 MethodChannel 등으로 네이티브 API를 직접 호출합니다.

- Android: `MobileAds.openDebugMenu(activity, adUnitId)` — [Google 공식 가이드](https://developers.google.com/ad-manager/mobile-ads-sdk/android/debug?hl=ko)
- iOS: `DebugOptionsViewController(adUnitID:)` — [Google 공식 가이드](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/debug?hl=ko)

메뉴에 사용할 광고 단위 ID는 운영 담당자에게 전달받은 값을 사용하세요. 확인 절차는 네이티브와 같습니다.

1. Debug Options Menu를 열 수 있는 테스트 앱을 빌드합니다.
2. 테스트 앱을 운영 담당자에게 전달합니다.
3. 운영 담당자가 테스트 기기에서 메뉴를 열고 광고를 요청해 입찰 광고 소스와 어댑터 상태를 확인합니다. 어댑터별 초기화 상태와 SDK 버전은 메뉴에서 **Open ad inspector**를 선택한 뒤 **Adapters** 탭에서 확인합니다. ad inspector는 테스트 기기로 등록된 기기에서만 열립니다. ([Android](https://developers.google.com/ad-manager/mobile-ads-sdk/android/ad-inspector/launch-ad-inspector), [iOS](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/ad-inspector/launch-ad-inspector))

운영 빌드에서는 디버그 메뉴 진입점을 노출하지 마세요.

## 6. 개인정보 신호와 nap mx 초기화

초기화는 앱 프로세스에서 한 번만 수행합니다. CMP·연령 설정 등 필요한 판단이 끝난 뒤, 첫 광고 요청 전에 호출하세요. `unspecified`는 동의가 아니라 “플러그인이 해당 값을 설정하지 않음”입니다.

```dart
import 'package:nap_mx_flutter/nap_mx_flutter.dart';

Future<void> initializeNapMx() async {
  // 발급값이 비어 있으면 네이티브 SDK를 호출하지 않습니다.
  if (mediaKey.isEmpty || bannerId.isEmpty || rewardedId.isEmpty) {
    throw StateError('nap mx 테스트 설정이 필요합니다.');
  }

  await NapMx.initialize(
    NapMxConfiguration(
      mediaKey: mediaKey,

      // 앱에서 사용할 모든 지면을 목록으로 등록합니다(v0.1.4 이상).
      additionalAdUnitIds: const [bannerId, rewardedId],

      // 아래 값은 예시 기본 동의가 아닙니다.
      // CMP 결과가 없으면 임의로 granted를 넣지 않습니다.
      privacy: const NapMxPrivacySettings(
        gdprConsent: NapMxConsentStatus.unspecified,
        usSaleConsent: NapMxConsentStatus.unspecified,
        childDirected: NapMxConsentStatus.unspecified,
        underAgeOfConsent: NapMxConsentStatus.unspecified,
      ),
      logLevel: NapMxLogLevel.error,

      // Android 전용 전역 테스트 모드입니다.
      // iOS에서는 true가 명시적 오류이므로 공용 코드는 false를 사용합니다.
      testMode: false,
    ),
  );
}
```

ATT 프롬프트는 플러그인이 자동으로 띄우지 않습니다. 앱이 요청 여부, 설명 문구와 호출 시점을 책임집니다. 동의 미설정을 동의로 바꾸지 마세요.

같은 설정으로 동시에 여러 번 초기화하면 하나의 작업을 공유합니다. 다른 Media Key나 설정으로 이미 초기화된 프로세스에서 다시 호출하면 실패하므로 광고 화면마다 초기화하지 마세요.

```dart
try {
  await initializeNapMx();
} on NapMxError catch (error) {
  // Media Key/AdUnit ID 원문은 로그에 남기지 않습니다.
  debugPrint('nap mx init failed: ${error.code} / ${error.message}');
  // 광고가 없는 상태로 앱의 본 기능은 계속 제공하도록 복구합니다.
}
```

### 같은 포맷의 여러 지면 등록 (v0.1.4 이상)

사용할 모든 지면 ID를 `additionalAdUnitIds: List<String>` 목록으로 등록하는 것을 권장합니다. 같은 포맷의 지면이 여러 개여도 모두 목록에 넣으면 되고, `adUnitIds` 맵은 생략할 수 있습니다. 기존 `adUnitIds` 맵도 계속 지원합니다.

```dart
await NapMx.initialize(NapMxConfiguration(
  mediaKey: mediaKey,
  additionalAdUnitIds: ['789', '790'], // 리워드 지면 A, B의 예시 ID
));

final placementB = NapMxFullscreenAdController(
  format: NapMxAdFormat.rewarded,
  adUnitId: '790',
);
await placementB.load();
// 화면 수명에 맞춰 placementB.dispose()를 호출하세요.
```

두 컬렉션을 합쳐 공백과 중복 ID를 제거한 뒤 네이티브 SDK 초기화에 전달합니다. iOS에서는 모든 ID가 정수 문자열이어야 합니다. 두 컬렉션이 모두 비었거나 빈 ID가 포함되면 `ArgumentError`가 발생합니다.

컨트롤러와 광고 뷰에는 사용할 ID를 명시하세요. `adUnitIdFor(format)`은 기존 맵만 조회하며 추가 목록에서 ID를 선택하지 않습니다. 초기화 후 재호출로 ID를 추가할 수 없으므로 사용할 지면을 처음에 모두 등록해야 합니다. 미등록 ID도 load 시 SDK로 전달되지만 실제 로드 성공 여부는 검증되지 않았습니다.

다음 단계: [광고 구현](/flutter/ad-formats)
