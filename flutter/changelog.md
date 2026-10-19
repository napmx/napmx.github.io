# 릴리즈 노트

Flutter 플러그인(`nap_mx_flutter`)은 Git 태그로 배포합니다. 설치할 때는 [설치와 초기화](/flutter/getting-started?id=2-플러그인-설치)의 `ref`에 아래 태그를 지정하세요. 네이티브 SDK의 변경 내용은 [Android 릴리즈 노트](/android/native/changelog)와 [iOS 릴리즈 노트](/ios/native/changelog)를 참고하세요.

---

## v0.1.7 (2026-10-19)

| 네이티브 구성 | 버전 |
|---|---|
| Android Core / BOM | 2.4.0 / 2026.10.04 (변경 없음) |
| iOS Mediation | 2.5.1 (변경 없음) |
| iOS Core | 1.3.0 (변경 없음) |

- Android·iOS의 `getSdkInfo()`가 반환하는 `pluginVersion`을 실제 플러그인 버전과 일치하도록 수정했습니다. `v0.1.5`와 `v0.1.6`은 `0.1.4`를 반환했습니다.
- CocoaPods에서 Privacy Manifest(`PrivacyInfo.xcprivacy`)를 소스 파일이 아닌 리소스 번들로만 포함하도록 수정했습니다.
- 플러그인 저장소 문서의 공식 가이드 링크가 실제 페이지로 연결되도록 수정했습니다.

---

## v0.1.6 (2026-10-19)

| 네이티브 구성 | 버전 |
|---|---|
| Android Core / BOM | **2.4.0** / **2026.10.04** |
| iOS Mediation | **2.5.1** |
| iOS Core | 1.3.0 (변경 없음) |

- 네이티브 SDK를 Android Core 2.4.0, iOS Mediation 2.5.1로 업데이트했습니다. iOS Core는 1.3.0을 유지하며 CocoaPods 고정 버전을 Swift Package Manager와 맞췄습니다.
- Google 광고 ID(`play-services-ads-identifier`) `18.2.0` 직접 선언을 제거했습니다. Android Core가 가져오는 API 21 호환 `18.0.0`이 적용되며, `minSdk` 23 이상 앱은 구형 Android의 `java.time.Duration` 크래시가 수정된 `18.3.0`을 선택할 수 있습니다.
- Android 어댑터 예제를 BOM `2026.10.04` 기준으로 갱신했습니다.

---

## v0.1.5 (2026-10-19)

- 인라인 광고(`NapMxAdView`)의 auto-load를 controller 이벤트 구독이 준비된 뒤에 시작하도록 수정했습니다.
- 취소된 인라인 요청과 iOS에서 늦게 도착한 광고 응답을 안전하게 무시·해제합니다.
- 전면 광고는 `dispose()`된 상태를 유지하고 `show()` 이후 재사용을 거부하며, 지연되거나 중복된 로드 콜백이 표시 중인 광고에 영향을 주지 않도록 수정했습니다. `show()` 이후 다음 광고에는 새 controller를 만드세요.
- 초기화 전에 auto-load가 실행되면 `NapMxAdView.build`에서 예외를 던지지 않고 `loadFailed`로 `not_initialized`를 전달합니다.
- `onViewCreated`에서 이미 `load()`를 호출했다면 auto-load를 건너뜁니다.
- `no_activity` / `no_view_controller`로 `show()`가 거절돼도 로드된 상태를 유지해 다시 `show()`할 수 있습니다.
- 네이티브 `dispose` 실패 뒤에 `dispose()`를 다시 호출해도 정상 완료됩니다.

---

## v0.1.4 (2026-10-19)

- 같은 포맷의 지면 여러 개를 등록할 수 있도록 `additionalAdUnitIds: List<String>`를 추가했습니다. 기존 `adUnitIds` 맵과 기본 ID 조회는 그대로 유지되며, 목록만으로 초기화할 수도 있습니다. [같은 포맷의 여러 지면 등록](/flutter/getting-started?id=같은-포맷의-여러-지면-등록-v014-이상)을 참고하세요.
- 두 컬렉션을 합쳐 중복을 제거한 ID를 Android와 iOS 초기화에 전달하며, Dart의 동일 설정 판정에도 추가 지면을 포함합니다.
- iOS에서 여러 포맷에 같은 ID를 등록하면 검증에 실패하던 문제를 수정했습니다.

---

## v0.1.3 (2026-10-08)

- AGP 8 호스트와 내장 Kotlin을 끈 AGP 9 호스트에서 Android Kotlin 컴파일이 실패하던 문제를 수정했습니다. 플러그인은 별도 buildscript classpath를 고정하지 않고 호스트의 AGP·Kotlin 플러그인 버전을 사용하며, AGP 9 내장 Kotlin도 계속 지원합니다.
- 예제 앱을 AGP 8.10.1 / Kotlin 2.2.10 / Gradle 8.11.1로 옮겼습니다.

---

## v0.1.2 (2026-09-22)

- 리워드 광고에서 네트워크가 닫힘 콜백 이후에 보상을 보고해도 앱에 전달되도록, 닫힌 전면 광고를 잠시 유지합니다. 보상 전달은 정확히 한 번만 일어납니다. 닫힘 직후 `dispose()`하면 이 유예가 끊기므로 [광고 구현](/flutter/ad-formats)의 순서를 지키세요.
- Android 인라인 광고 리스너가 GC되어 배너 이벤트가 조용히 끊기던 문제를 수정했습니다.
- Android 배너·네이티브·인라인 동영상 View에 호스트 Activity의 `onResume` / `onPause`를 전달합니다(네이티브 SDK 요구사항).
- iOS 리워드 포맷에서도 `customParams`가 전달된다는 점을 문서에 바로잡았습니다.
- Android SDK가 요구하는 `play-services-ads` 25.2.0 상한을 문서화했습니다.

---

## v0.1.1 (2026-09-21)

- 매체사 대상의 Android·iOS 연동 가이드를 추가했습니다.
- 플랫폼별 초기화, 개인정보, 수명주기, 이벤트, 오류, 보상 처리를 명확히 정리했습니다.
- PlatformView 채널 실패를 `NapMxError`로 통일하고 no-fill/timeout 구분을 유지합니다.

---

## v0.1.0 (2026-09-21)

- 초기화, 개인정보, 이벤트, 오류, 요청 API를 타입으로 제공합니다.
- 배너·네이티브·인라인 동영상 네이티브 View를 추가했습니다.
- 전면·아웃스트림 동영상·리워드 controller를 추가했습니다.
- Android SDK 2.3.0, iOS SDK 2.5.0을 연동했습니다.
- Material 3 예제 앱, 테스트, CI와 공개 연동 가이드를 추가했습니다.
