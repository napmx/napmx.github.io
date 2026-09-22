# Flutter 이벤트·오류·테스트

운영 코드에서는 개별 controller의 이벤트로 화면 상태를 갱신하고, `NapMx.events` 전역 스트림은 공통 분석·진단에만 사용하세요. SDK가 제공하지 않은 네트워크 이름이나 어댑터 버전을 추정해서 채우지 않습니다.

## 이벤트 수신

```dart
late final StreamSubscription<NapMxEvent> allAdEvents;

void startAdMonitoring() {
  allAdEvents = NapMx.events.listen((event) {
    debugPrint(
      'type=${event.type.name}, '
      'format=${event.format?.name}, '
      // 네이티브 SDK가 실제 보고하지 않으면 null입니다.
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
| `shown` | SDK 표시 콜백 | 노출 상태 기록 |
| `showFailed` | 표시 실패 | controller 해제 후 필요 시 새 요청 |
| `clicked` | SDK 클릭 콜백 | 분석만 기록, 클릭 자동화 금지 |
| `completed` | 동영상 완료 | 리워드 지급 근거로 사용하지 않음 |
| `skipped` | 동영상 건너뜀 | 화면 상태 복구 |
| `rewarded` | 실제 SDK 보상 콜백 | `transactionId` 기준 1회 지급 |
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
    // 광고 재고가 없는 정상적인 상황일 수 있습니다.
    // 성공으로 바꾸거나 즉시 무한 재요청하지 않습니다.
    hideAdSlotTemporarily();
  } else if (error.isTimeout) {
    showContentWithoutAd();
  } else {
    // message 문자열 대신 안정된 code로 분기합니다.
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
| `not_initialized` | 초기화 완료 전 광고 요청 |
| `already_loading`, `duplicate_request` | 같은 controller에 동시 요청 |
| `not_loaded` | 로드 성공 전 show 또는 네이티브 객체 소실 |
| `load_failed` | `nativeCode`, 선택 어댑터와 서버 설정 확인 |
| `timeout` | 네트워크 상태와 30초 기본 제한 확인 |
| `no_activity`, `no_view_controller` | 광고를 표시할 활성 화면 없음 |
| `show_failed` | load 성공 뒤 같은 controller인지 확인 |
| `cancelled`, `disposed` | 취소/해제된 요청을 다시 사용함 |
| `test_mode_unsupported` | iOS에서 Android 전용 전역 테스트 모드를 요청함 |
| `mediation_configuration_unsupported` | iOS에서 Android 전용 mediation 맵을 전달함 |
| `unsupported_privacy` | Android Core에 없는 under-age 값을 명시함 |

Android `0x8000000x` 계열은 [Android 오류 코드](/android/native/error-codes), iOS `-1`~`-8`은 [iOS SDK 가이드](/ios/native/getting-started)의 원본 표를 확인하세요.

## Sample 앱 실행

Sample은 플러그인의 공개 API만 사용하며 Mock 광고를 사용하지 않습니다. 테스트 설정이 비어 있으면 요청 버튼을 막습니다.

```bash
git clone --branch v0.1.2 --depth 1 \
  https://github.com/Nasmedia-Tech/nap_mx_flutter.git
cd nap_mx_flutter/example

# macOS/Linux
cp config/example.json config/local.json

# Windows PowerShell
Copy-Item config/example.json config/local.json
```

`config/local.json`의 placeholder를 발급받은 테스트 값으로 바꾼 뒤 실행합니다.

```bash
flutter pub get
flutter run --dart-define-from-file=config/local.json
```

Sample에서 확인할 수 있는 항목:

- 포맷별 load/show/dispose 상태와 버튼 활성화
- 오류 code와 native code
- SDK가 실제 전달한 네트워크 이름
- 요청 소요 시간
- reward `transactionId`
- 시스템 다크 모드, SafeArea, 작은 화면 스크롤과 글자 확대

Sample 빌드 성공은 실제 광고 응답 성공의 증거가 아닙니다. 실기기 검증 중 광고 자체를 자동 클릭하지 마세요.

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

공개 저장소 CI는 등록된 self-hosted macOS ARM64 runner에서 Android와 iOS를 모두 검증합니다. 현재 자동 검증 범위는 다음과 같습니다.

| 검증 | 결과 | 의미 |
|---|---|---|
| Dart format/analyze/test | PASS | 공개 API와 핵심 상태 테스트 통과 |
| `pub publish --dry-run` | PASS | 패키지 구성 점검, 실제 pub.dev 배포는 하지 않음 |
| Android debug/release 빌드 | PASS | 네이티브 의존성과 플러그인 컴파일 성공 |
| iOS Simulator 빌드 | PASS | Swift/SPM 또는 Pods 연결과 컴파일 성공 |
| Android 실제 광고/보상 | NOT_RUN | 발급 테스트 지면과 실기기 필요 |
| iOS 실제 광고/보상 | NOT_RUN | 발급 테스트 지면과 실기기 필요 |

최신 결과는 [GitHub Actions](https://github.com/Nasmedia-Tech/nap_mx_flutter/actions)에서 확인할 수 있습니다.

## 출시 전 체크리스트

- [ ] Android/iOS 각각 올바른 Media Key와 AdUnit ID를 사용했다.
- [ ] 운영 ID, 인증서, 서명키가 Git과 로그에 노출되지 않았다.
- [ ] 실제 사용하는 선택 어댑터만 설치했다.
- [ ] 네트워크별 App ID와 iOS 네이티브 초기화를 완료했다.
- [ ] 개인정보 선택을 SDK/네트워크 초기화보다 먼저 확정했다.
- [ ] ATT 요청 여부·문구·호출 시점을 앱 정책에 맞게 결정했다.
- [ ] Privacy Manifest, SKAdNetwork와 스토어 개인정보 항목을 검토했다.
- [ ] 작은 화면과 글자 확대에서 광고 View가 잘리지 않는다.
- [ ] 전면 광고가 활성 화면에서 `load → show` 순서로 동작한다.
- [ ] 보상은 `rewarded` 콜백에서만 지급하고 `transactionId` 중복을 차단한다.
- [ ] S2S를 사용하면 같은 `transaction_id` 재수신을 무적립 처리한다.
- [ ] 화면 재진입·백그라운드 복귀·해제 뒤 늦은 콜백을 확인했다.
- [ ] no-fill/timeout에 무한 재요청하지 않는다.
- [ ] Android/iOS 실기기에서 테스트 지면으로 최종 확인했다.

문제를 문의할 때는 플랫폼, 플러그인 버전, 네이티브 SDK 버전, 포맷, 오류 code/nativeCode와 재현 순서를 전달하세요. Media Key와 AdUnit ID 원문은 공개 이슈에 첨부하지 마세요.
