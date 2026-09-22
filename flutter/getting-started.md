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

현재 `pub.dev`에는 배포하지 않습니다. 재현 가능한 빌드를 위해 `main` 브랜치 대신 검증된 태그를 지정합니다.

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  nap_mx_flutter:
    git:
      url: https://github.com/Nasmedia-Tech/nap_mx_flutter.git
      ref: v0.1.2 # 검증된 태그를 고정합니다.
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

플러그인이 nap mx Core를 가져옵니다. 매체 앱은 사용하는 네트워크 어댑터만 추가합니다.

```kotlin
// android/app/build.gradle.kts
dependencies {
    // nap mx 모듈을 공식 검증 조합으로 정렬합니다.
    implementation(platform("io.github.nasmedia-tech:admixer-bom:2026.09.03"))

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

AdFit, Pangle, Teads를 사용할 때만 해당 Maven 저장소를 추가합니다.

```kotlin
// android/settings.gradle.kts
dependencyResolutionManagement {
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

## 5. iOS 설정

플러그인은 `AdMixerMediation` 2.5.0과 `AdMixer` 1.3.0을 고정합니다. Flutter 프로젝트가 Swift Package Manager를 사용하면 Xcode Package Dependencies에 필요한 공식 어댑터만 추가합니다. CocoaPods 프로젝트라면 `ios/Podfile`에 필요한 Pod만 추가합니다.

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

## 6. 개인정보 신호와 nap mx 초기화

초기화는 앱 프로세스에서 한 번만 수행합니다. CMP·연령 설정 등 필요한 판단이 끝난 뒤, 첫 광고 요청 전에 호출하세요. `unspecified`는 동의가 아니라 “플러그인이 해당 값을 설정하지 않음”입니다.

```dart
import 'package:nap_mx_flutter/nap_mx_flutter.dart';

Future<void> initializeNapMx() async {
  // 발급값이 비어 있으면 네이티브 SDK를 호출하지 않습니다.
  if (mediaKey.isEmpty || rewardedId.isEmpty) {
    throw StateError('nap mx 테스트 설정이 필요합니다.');
  }

  await NapMx.initialize(
    NapMxConfiguration(
      mediaKey: mediaKey,

      // 앱에서 실제로 사용하는 포맷만 등록합니다.
      adUnitIds: const {
        NapMxAdFormat.banner: bannerId,
        NapMxAdFormat.rewarded: rewardedId,
      },

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

다음 단계: [광고 구현](/flutter/ad-formats)
