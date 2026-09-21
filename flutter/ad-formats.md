# Flutter 광고 구현

초기화가 성공한 뒤 광고를 요청합니다. 화면 안에 배치하는 광고와 전체 화면 광고는 서로 다른 수명주기를 사용합니다.

| 구분 | 포맷 | API | 기본 순서 |
|---|---|---|---|
| View 광고 | 배너, 네이티브, 인스트림 동영상 | `NapMxAdView` | View 생성 → `load` → `dispose` |
| 전체 화면 | 전면, 아웃스트림 동영상, 리워드 | `NapMxFullscreenAdController` | controller 생성 → `load` → `show` → `dispose` |

## 배너·네이티브·인스트림 동영상

세 포맷은 실제 네이티브 SDK View를 Flutter `PlatformView`로 표시합니다. 다음 예제는 생성·로드·오류 처리·해제를 모두 포함합니다.

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:nap_mx_flutter/nap_mx_flutter.dart';

class BannerSlot extends StatefulWidget {
  const BannerSlot({required this.adUnitId, super.key});

  final String adUnitId;

  @override
  State<BannerSlot> createState() => _BannerSlotState();
}

class _BannerSlotState extends State<BannerSlot> {
  NapMxAdViewController? _controller;
  StreamSubscription<NapMxEvent>? _events;
  String _status = '광고 준비 중';

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        // ListView 안에서는 실제 광고 높이를 명시해 무한 높이 오류를 막습니다.
        SizedBox(
          height: 50,
          child: NapMxAdView(
            format: NapMxAdFormat.banner,
            adUnitId: widget.adUnitId,
            height: 50,
            autoLoad: false,
            onViewCreated: (controller) async {
              _controller = controller;

              // 이 View에서 발생한 이벤트만 수신합니다.
              _events = controller.events.listen((event) {
                if (!mounted) return;
                setState(() => _status = event.type.name);
              });

              try {
                // PlatformView 생성이 끝난 뒤 한 번만 호출합니다.
                await controller.load();
              } on NapMxError catch (error) {
                if (!mounted) return;
                setState(() => _status = '실패: ${error.code}');
              }
            },
          ),
        ),
        Text(_status),
      ],
    );
  }

  @override
  void dispose() {
    // State.dispose는 async가 아니므로 정리를 시작만 합니다.
    // controller 폐기 이후 들어오는 네이티브 콜백은 무시됩니다.
    unawaited(_events?.cancel());
    unawaited(_controller?.dispose());
    super.dispose();
  }
}
```

다른 View 포맷은 `format`과 광고 슬롯 높이를 바꿉니다.

```dart
// 네이티브 광고: SDK 템플릿의 CTA/AdChoices/미디어 영역을 보존합니다.
NapMxAdView(
  format: NapMxAdFormat.native,
  adUnitId: nativeId,
  height: 360,
)

// 인스트림(인라인) 동영상: 스크롤 영역의 실제 가용 폭과 높이를 지정합니다.
NapMxAdView(
  format: NapMxAdFormat.inlineVideo,
  adUnitId: inlineVideoId,
  height: 180,
)
```

네이티브 광고 자산을 Flutter 위젯으로 임의 재조합하면 SDK의 노출·클릭 등록을 우회할 수 있습니다. 플러그인이 제공하는 네이티브 템플릿을 사용하고 작은 화면·다크 모드·글자 확대에서 CTA, `광고/Ad` 표시와 AdChoices가 잘리지 않는지 확인하세요.

화면은 유지하지만 진행 중 요청만 중단하려면 `controller.cancel()`을 호출할 수 있습니다. 화면에서 제거할 때는 반드시 `dispose()`합니다.

## 전면·아웃스트림 동영상

controller 하나는 요청 하나만 소유합니다. 로드된 controller를 여러 화면이나 여러 요청에서 공유하지 마세요.

```dart
import 'dart:async';

import 'package:flutter/foundation.dart';
import 'package:nap_mx_flutter/nap_mx_flutter.dart';

class InterstitialOwner {
  NapMxFullscreenAdController? _ad;
  StreamSubscription<NapMxEvent>? _events;

  Future<void> load(String adUnitId) async {
    // 이전 화면/요청이 남아 있으면 먼저 해제합니다.
    await dispose();

    final ad = NapMxFullscreenAdController(
      format: NapMxAdFormat.interstitial,
      adUnitId: adUnitId,
      loadTimeout: const Duration(seconds: 30),
    );
    _ad = ad;

    _events = ad.events.listen((event) {
      if (event.type == NapMxEventType.loadFailed) {
        debugPrint('load failed: ${event.error?.code}');
      }
    });

    // 네이티브 SDK가 성공 또는 실패를 반환할 때 완료됩니다.
    await ad.load();
  }

  Future<void> show() async {
    final ad = _ad;
    if (ad == null || ad.state != NapMxAdState.loaded) {
      throw StateError('광고가 아직 로드되지 않았습니다.');
    }

    // 사용자 액션 직후, 앱의 현재 화면이 활성 상태일 때 표시합니다.
    await ad.show();
  }

  Future<void> dispose() async {
    await _events?.cancel();
    _events = null;
    await _ad?.dispose();
    _ad = null;
  }
}
```

아웃스트림 전면 동영상은 `NapMxAdFormat.interstitialVideo`를 사용하고 같은 수명주기를 지킵니다. 앱이 백그라운드이거나 Android Activity/iOS ViewController가 활성화되기 전에는 `show()`하지 마세요.

## 리워드 동영상

보상은 `show()` 성공, 영상 완료 추정, 화면 닫힘으로 지급하지 않습니다. 실제 SDK의 `NapMxEventType.rewarded` 이벤트에서만 처리하고 `transactionId`를 지급 원장의 고유 키로 저장하세요.

```dart
import 'dart:async';

import 'package:nap_mx_flutter/nap_mx_flutter.dart';

Future<void> showRewarded(String adUnitId) async {
  final rewarded = NapMxFullscreenAdController(
    format: NapMxAdFormat.rewarded,
    adUnitId: adUnitId,
  );

  // show()는 표시 성공 시 완료되므로 closed까지 구독을 유지합니다.
  final closed = Completer<void>();
  final subscription = rewarded.events.listen((event) {
    if (event.type == NapMxEventType.rewarded && event.reward != null) {
      final transactionId = event.reward!.transactionId;
      if (transactionId.isNotEmpty) {
        // 서버/영속 저장소에서 transactionId UNIQUE를 보장합니다.
        // 앱 메모리 Set만으로는 재실행·다중 기기의 중복 지급을 막지 못합니다.
        unawaited(rewardRepository.grantOnce(transactionId));
      }
    }

    if ((event.type == NapMxEventType.closed ||
            event.type == NapMxEventType.showFailed) &&
        !closed.isCompleted) {
      closed.complete();
    }
  });

  try {
    await rewarded.load();
    await rewarded.show();
    await closed.future;
  } finally {
    await subscription.cancel();
    await rewarded.dispose();
  }
}
```

### S2S 리워드 검증

S2S Reward Callback을 사용하면 nap mx 서버가 매체 서버의 등록된 URL로 지급 정보를 전달합니다. 앱 콜백과 서버 콜백에 같은 `transaction_id`가 전달되므로 `(adunit_id, transaction_id)`를 고유 키로 두고 재시도·중복 수신을 무적립 처리하세요.

- Android 등록·파라미터·재시도 정책: [Android 리워드 S2S 가이드](/android/native/rewarded-video?id=s2s-reward-callback-서버-간-리워드-검증)
- iOS 등록·파라미터: [iOS 리워드 S2S 가이드](/ios/native/rewarded-video?id=s2s-reward-callback-선택사항)

Android에서는 필요한 경우 요청별 값을 `customParams`로 전달할 수 있습니다. `transaction_id`는 SDK 예약 키이므로 직접 덮어쓰지 마세요. iOS 플러그인은 현재 이 맵을 사용하지 않으므로 공통 로직이 이 값에 의존해서는 안 됩니다.

```dart
await rewarded.load(
  customParams: {
    // 서버가 사용자를 대조하는 매체 정의 값의 예시입니다.
    // 인증 토큰이나 민감정보 원문을 넣지 마세요.
    'user_id': currentUser.rewardMemberId,
  },
);
```

리워드 금액과 종류는 플러그인이 만들지 않습니다. 실제 지급 정책과 S2S 콜백 URL 등록은 매체 서버 담당자와 nap mx 운영 담당자가 함께 확정합니다.

## 화면과 앱 수명주기

- 화면을 나갈 때 이벤트 구독과 controller를 함께 해제합니다.
- 화면 재진입 시 새 controller와 새 요청을 만듭니다.
- Android Activity 재생성 뒤 이전 화면의 controller를 새 화면에서 재사용하지 않습니다.
- iOS foreground-active Scene이 없을 때 전면 광고를 표시하지 않습니다.
- no-fill/timeout에 즉시 무한 재요청하지 말고 최대 횟수와 backoff를 둡니다.
- 실제 광고 클릭을 자동화하지 않습니다.

다음 단계: [이벤트·오류·테스트](/flutter/events-and-testing)
