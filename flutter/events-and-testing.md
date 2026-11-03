# Flutter 이벤트·오류·테스트

화면 상태는 각 controller의 이벤트로 갱신하고, `NapMx.events` 전역 스트림은 공통 로그나 분석에 사용하세요.

## 이벤트 수신

```dart
late final StreamSubscription<NapMxEvent> allAdEvents;

void startAdMonitoring() {
  allAdEvents = NapMx.events.listen((event) {
    debugPrint(
      'type=${event.type.name}, '
      'format=${event.format?.name}, '
      // SDK가 응답 네트워크를 알려 주지 않으면 null입니다.
      'network=${event.network ?? "not-reported"}, '
      'elapsedMs=${event.elapsedMilliseconds}, '
      'error=${event.error?.code}',
    );
  });
}

Future<void> stopAdMonitoring() => allAdEvents.cancel();
```

| 이벤트 | 의미 | 권장 처리 |
|---|---|---|
| `initializationSucceeded` | 네이티브 SDK 초기화 완료 | 광고 UI 활성화 |
| `loadStarted` | 요청 시작 | 로딩 상태 표시 |
| `loaded` | 표시 가능한 광고 로드 | Show 버튼 활성화 |
| `loadFailed` | 로드 실패 | 오류 기록 후 앱 본문 복구 |
| `shown` | SDK 표시 콜백. iOS 배너·네이티브는 광고 영역이 50% 이상 보일 때, nap mx 인라인 동영상은 실제 재생이 시작될 때 전달 (`v0.1.8` 이상) | 노출 상태 기록 |
| `showFailed` | 표시 실패. 인라인 뷰도 전달하며, 전면 광고가 화면에 뜨기 전 실패하면 플러그인이 광고를 바로 해제 (`v0.1.8` 이상) | controller 해제 후 필요 시 새 요청 |
| `clicked` | SDK 클릭 콜백 | 클릭 기록 |
| `completed` | 동영상 완료 | 리워드 지급 근거로 사용하지 않음 |
| `skipped` | 동영상 건너뜀 | 화면 상태 복구 |
| `rewarded` | SDK 보상 콜백 | `transactionId` 기준 1회 지급 |
| `closed` | 전체 화면 광고 닫힘 | 원래 화면 상태 복구 |
| `cancelled` | 요청 취소 | 필요 시 새 controller 생성 |
| `disposed` | 네이티브 참조 해제 | controller 재사용 금지 |

SDK/플러그인 버전은 다음처럼 확인합니다.

```dart
final info = await NapMx.getSdkInfo();
debugPrint('plugin=${info.pluginVersion}');
debugPrint('native=${info.sdkVersion ?? "not-reported"}');
debugPrint('adapters=${info.adapterVersions ?? "not-reported"}');
```

## 오류 처리

네이티브 실패는 `NapMxError`, 잘못된 호출 순서는 `StateError`로 전달될 수 있습니다. no-fill과 timeout은 별도 플래그로 판단합니다.

```dart
try {
  await controller.load();
} on NapMxError catch (error) {
  if (error.isNoFill) {
    // 보여 줄 광고가 없다는 뜻입니다. 바로 재요청하지 말고 슬롯을 숨깁니다.
    hideAdSlotTemporarily();
  } else if (error.isTimeout) {
    showContentWithoutAd();
  } else {
    // message는 바뀔 수 있으니 code로 분기합니다.
    reportAdError(error.code, error.nativeCode);
  }
} on StateError catch (error) {
  // 초기화 전 load, load 전 show, dispose 후 재사용 같은 앱 코드 오류입니다.
  debugPrint('invalid ad state: $error');
}
```

| 오류 코드 | 확인 사항 |
|---|---|
| `invalid_configuration`, `invalid_ad_unit` | 빈 키/ID, iOS에서 숫자가 아닌 값 |
| `already_initialized` | 다른 설정으로 중복 초기화 |
| `not_initialized` | 초기화 완료 전 광고 요청 (인라인 auto-load도 예외 대신 `loadFailed`로 전달) |
| `already_loading`, `duplicate_request` | 같은 controller에 동시 요청 |
| `not_loaded` | 로드 성공 전 show 또는 네이티브 객체 소실 |
| `load_failed` | `nativeCode`, 선택 어댑터와 서버 설정 확인 |
| `timeout` | 네트워크 상태와 30초 기본 제한 확인 |
| `no_activity`, `no_view_controller` | 광고를 표시할 활성 화면 없음 (로드된 상태가 유지되므로 화면 복귀 후 `show()` 재시도 가능) |
| `show_failed` | load 성공 뒤 같은 controller인지 확인 |
| `cancelled`, `disposed` | 취소/해제된 요청을 다시 사용함 |
| `test_mode_unsupported` | iOS에서 Android 전용 전역 테스트 모드나 `testDeviceIds`를 요청함 |
| `mediation_configuration_unsupported` | iOS에서 Android 전용 mediation 맵을 전달함 |
| `unsupported_privacy` | Android Core에 없는 under-age 값을, iOS에서 `usPrivacyString`을 명시함 |

Android 공통 코드(`-1`~`-31`, 예: no-fill `-20`, timeout `-22`)는 [Android 오류 코드](/android/native/error-codes), iOS `-1`~`-8`은 [iOS SDK 가이드](/ios/native/getting-started)의 원본 표를 확인하세요.

## Sample 앱 실행

테스트 값을 넣기 전에는 Sample 앱의 광고 요청 버튼이 비활성화됩니다.

```bash
git clone --branch v0.1.9 --depth 1 \
  https://github.com/Nasmedia-Tech/nap_mx_flutter.git
cd nap_mx_flutter/example

# macOS/Linux
cp config/example.json config/local.json

# Windows PowerShell
Copy-Item config/example.json config/local.json
```

`config/local.json`의 예시 값을 발급받은 테스트 값으로 바꾼 뒤 실행합니다.

```bash
flutter pub get
flutter run --dart-define-from-file=config/local.json
```

Sample에서 확인할 수 있는 항목:

- 포맷별 load/show/dispose 상태와 버튼 활성화
- 오류 code와 native code
- SDK가 알려 준 응답 네트워크
- 요청 소요 시간
- reward `transactionId`
- 시스템 다크 모드, SafeArea, 작은 화면 스크롤과 글자 확대

테스트 중에도 광고를 자동으로 클릭하지 마세요. 무효 트래픽으로 처리될 수 있습니다.

## 자동 검증 명령

플러그인 루트에서 다음 명령을 실행합니다.

```bash
flutter pub get
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
dart pub publish --dry-run

cd example
flutter pub get
flutter test
flutter build apk --debug
flutter build apk --release

# macOS/Xcode 환경
flutter build ios --simulator --no-codesign
```

플러그인 저장소의 CI는 self-hosted macOS(ARM64) 러너에서 다음 항목을 실행합니다(Flutter 3.44.9).

| 항목 | 내용 |
|---|---|
| Dart format·analyze·test | 공개 API와 상태 처리 테스트 |
| `pub publish --dry-run` | 패키지 구성 점검 (pub.dev에는 배포하지 않음) |
| Android debug/release 빌드 | AGP 8.10.1 / Gradle 8.11.1, Kotlin 2.1.0·2.2.10, R8 포함 |
| Android 플러그인 단위 테스트 | Kotlin 2.1.0·2.2.10 |
| iOS 시뮬레이터 빌드 | Swift Package Manager |
| iOS 플러그인 단위 테스트 | XCTest, iOS 시뮬레이터 |

결과는 [GitHub Actions](https://github.com/Nasmedia-Tech/nap_mx_flutter/actions)에서 볼 수 있습니다.

AGP 9.0.1 / Gradle 9.1.0은 Flutter 앱이 아닌 별도 Android 프로젝트에서 플러그인 debug/release 빌드와 단위 테스트로 확인했습니다.

| Kotlin 구성 | 결과 |
|---|---|
| 내장 Kotlin 2.2.10 사용 | 통과 |
| 내장 Kotlin·새 DSL 끔, 외부 Kotlin 플러그인 2.3.20 | 통과 |

다음 동작은 Android와 iOS 사이에 차이가 있는지 확인하고 있습니다.

- 동영상 건너뛰기(`skipped`) 콜백이 오는 조건
- 광고 클릭 후 앱으로 돌아왔을 때 재생·노출 이벤트가 다시 오는지, 그 경우 서버에서 중복 집계되는지

이런 증상이 보이면 SDK 버전과 이벤트 순서를 정리해 [운영 담당자](mailto:nap_mx@nasmedia.co.kr)에게 알려 주세요.

## 출시 전 체크리스트

- [ ] Android/iOS 각각 올바른 Media Key와 AdUnit ID를 사용했다.
- [ ] 운영 ID, 인증서, 서명키가 Git과 로그에 노출되지 않았다.
- [ ] 사용하는 선택 어댑터만 설치했다.
- [ ] 네트워크별 App ID와 iOS 네이티브 초기화를 완료했다.
- [ ] Google을 사용한다면 입찰 광고 소스를 호환 버전으로 추가하고(Unity Ads는 `unity-ads` SDK 포함), Debug Options Menu → Open ad inspector → Adapters 탭에서 어댑터 초기화 상태를 확인했다.
- [ ] 개인정보 선택을 SDK/네트워크 초기화보다 먼저 확정했다.
- [ ] ATT 요청 여부·문구·호출 시점을 앱 정책에 맞게 결정했다.
- [ ] Privacy Manifest, SKAdNetwork와 스토어 개인정보 항목을 검토했다.
- [ ] 작은 화면과 글자 확대에서 광고 View가 잘리지 않는다.
- [ ] 전면 광고가 활성 화면에서 `load → show` 순서로 동작한다.
- [ ] 보상은 `rewarded` 콜백에서만 지급하고 `transactionId` 중복을 차단한다.
- [ ] S2S를 사용하면 같은 `transaction_id`가 다시 와도 한 번만 지급한다.
- [ ] 화면 재진입·백그라운드 복귀·해제 뒤에도 광고가 정상 동작한다.
- [ ] no-fill/timeout 뒤 바로 반복 재요청하지 않는다.
- [ ] Android/iOS 실기기에서 테스트 지면으로 최종 확인했다.

문제를 문의할 때는 플랫폼, 플러그인 버전, 네이티브 SDK 버전, 포맷, 오류 code/nativeCode와 재현 순서를 전달하세요. Media Key와 AdUnit ID 원문은 공개 이슈에 첨부하지 마세요.
