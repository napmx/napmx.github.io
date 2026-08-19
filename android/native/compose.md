# Jetpack Compose 연동

Compose 앱에서 nap mx 광고를 연동하는 방법입니다. 코어 SDK는 View 기반이므로, Compose에서는 **생명주기 연결**과 **해제 시점**을 직접 관리해야 합니다. 이를 자동 처리하는 **공식 헬퍼 모듈 `admixer-compose`** 를 제공합니다.

> ℹ️ 기본 설정(미디어 키/초기화/ProGuard)은 [SDK 시작하기](getting-started.md)를 먼저 완료하세요.

---

## 의존성 추가

**방법 A — BOM 사용 (권장)** — 버전 생략, BOM이 관리:

```gradle
dependencies {
    implementation platform('io.github.nasmedia-tech:admixer-bom:2026.08.01')
    implementation 'io.github.nasmedia-tech:admixer-ssp'        // 코어(필수)
    implementation 'io.github.nasmedia-tech:admixer-compose'    // Compose 헬퍼
    // 사용하는 어댑터 모듈 + play-services-ads-identifier 등은 시작하기 가이드 참고
}
```

**방법 B — 개별 버전 지정** (작성 시점 최신 배포 버전 기준):

```gradle
dependencies {
    implementation 'io.github.nasmedia-tech:admixer-ssp:2.2.0'        // 코어(필수)
    implementation 'io.github.nasmedia-tech:admixer-compose:2.0.2'    // Compose 헬퍼
    // 사용하는 어댑터 모듈 + play-services-ads-identifier 등은 시작하기 가이드 참고
}
```

> ℹ️ **`admixer-compose`의 버전 번호는 코어(`admixer-ssp`)와 다릅니다.** 모듈별로 변경이 있을 때만 개별 배포되므로 두 버전은 일치하지 않는 것이 정상입니다(예: 코어 `2.2.0` + Compose 헬퍼 `2.0.2`).
> 이 때문에 **방법 A(BOM)를 권장**합니다 — 버전을 생략하면 BOM이 해당 릴리스에서 검증된 멤버 버전으로 자동 고정하므로, 모듈별로 버전을 직접 맞추다 생기는 불일치·구버전 혼용을 예방할 수 있습니다. 개별 지정 방식을 쓴다면 [Maven Central](https://central.sonatype.com/namespace/io.github.nasmedia-tech)에서 각 모듈의 최신 버전을 확인해 갱신하세요.

---

## 배너 — `AdMixerBanner`

`AndroidView` 호스팅·`stop()` 해제·생명주기 연결·도착 콜백 가드를 헬퍼가 자동 처리합니다.

```kotlin
import com.nasmedia.admixerssp.compose.AdMixerBanner

@Composable
fun MyScreen() {
    AdMixerBanner(
        adUnitId = MyApplication.ADUNIT_ID_BANNER,
        modifier = Modifier.fillMaxWidth(),
    )
}
```

내부 동작:
- 컴포지션 진입 시 `AMMBannerView` 생성 + `loadAd()`, 이탈 시 `DisposableEffect.onDispose`에서 `stop()`.
- `ON_RESUME/ON_PAUSE`를 `adView.onResume()/onPause()`로 자동 연결 → **백그라운드 갱신/영상 재생 정지**.
- dispose 이후 도착하는 콜백은 무시(가드) → 스크롤 아웃/슬롯 재사용 시 상태 오염 방지.

리스너가 필요하면:

```kotlin
val listener = remember {
    object : AdListener() {
        override fun onReceivedAd(networkType: AdNetworkType, ad: Any) {
            // networkType로 switch: switch(networkType){ case PANGLE: ... }
        }
        override fun onFailedToReceiveAd(code: Int, msg: String?) {}
        override fun onAdDisplayed() {}
        override fun onAdClicked() {}
    }
}
AdMixerBanner(adUnitId = "...", listener = listener)
```

---

## 네이티브 — `AdMixerNativeAd`

네이티브 광고는 **레이아웃 바인더(`NativeAdViewBinder`)** 로 제목/이미지/CTA 등을 매핑합니다. 헬퍼가 `loadAd()` → AndroidView 부착 시 자동 렌더링 → 이탈 시 `stop()`까지 자동 처리합니다(별도 `showAd()` 호출 불필요).

```kotlin
import com.nasmedia.admixerssp.compose.AdMixerNativeAd
import com.nasmedia.admixerssp.common.nativeads.NativeAdViewBinder

@Composable
fun NativeSlot() {
    val binder = remember {
        NativeAdViewBinder.Builder(R.layout.item_native_ad)
            .setTitleId(R.id.nap_mx_tv_title)
            .setIconImageId(R.id.nap_mx_iv_icon)
            .setAdvertiserId(R.id.nap_mx_tv_adv)
            .setDescriptionId(R.id.nap_mx_tv_desc)
            .setMainViewId(R.id.nap_mx_iv_main)
            .setCtaId(R.id.nap_mx_btn_cta)
            .setAdChoicesPosition(AdChoicesPosition.RIGHT_TOP) // ✅ 선택 — AdChoices 모서리, 기본 RIGHT_TOP
            .build()
    }
    AdMixerNativeAd(
        adUnitId = MyApplication.ADUNIT_ID_NATIVE,
        binder = binder,
        modifier = Modifier.fillMaxWidth(),
    )
}
```

> ℹ️ View ID는 v2.0.0에서 `nap_mx_` prefix가 붙습니다. 자세한 내용은 [마이그레이션 Step 7](migration.md)을 참고하세요.

> ℹ️ **AdChoices(광고 정보 고지)** — SDK가 자동 오버레이하므로 레이아웃 슬롯이 필요 없습니다. 위치는 `setAdChoicesPosition(AdChoicesPosition.LEFT_TOP)`으로 4개 모서리 중 하나를 **요청**할 수 있습니다(기본 `RIGHT_TOP`). 다만 일부 네트워크는 지정 위치를 무시하고 자체 규칙으로 배치할 수 있습니다. 자세한 내용은 [네이티브 가이드](native-ad.md#광고-정보-고지adchoices-위치-지정)를 참고하세요.

---

## 전면 — `rememberInterstitialAd`

전면·리워드는 **Activity 노출형**이라 화면에 배치하는 컴포저블이 아니라, `remember*`가 반환하는 **state-holder 핸들**로 제어합니다. 헬퍼가 매니저를 컴포지션당 1개만 생성하고(중복 콜백 방지), 이탈 시 `stop()`으로 정식 해제합니다.

```kotlin
import com.nasmedia.admixerssp.compose.rememberInterstitialAd

@Composable
fun MyScreen() {
    val interstitial = rememberInterstitialAd(
        adUnitId = MyApplication.ADUNIT_ID_INTERSTITIAL,
        // autoShow = false (기본): 수신 후 원하는 시점에 show() 직접 호출
    )

    // 화면 진입 시 미리 로드
    LaunchedEffect(Unit) { interstitial.load() }

    Button(onClick = { if (interstitial.isLoaded) interstitial.show() }) {
        Text("전면 광고 보기")
    }
}
```

- `load()` 비동기 로드 / `show()` 노출 / `cancelLoad()` 진행 중 로드만 취소 / `isLoaded` 수신 여부.
- 로드 즉시 노출하려면 `rememberInterstitialAd(adUnitId, autoShow = true)`. 내부적으로 수신 시 `show()`만 호출하며 `start*`를 재호출하지 않습니다.

> 🚨 **수신 콜백에서 `start*`(로드+자동노출)를 직접 호출하지 마세요.** 매 수신마다 재로드되어 무한 루프가 발생합니다. 노출은 항상 `show()`(=`showAd()`)로 하며, 헬퍼는 이 규약을 강제합니다.

---

## 리워드 — `rememberRewardVideoAd`

```kotlin
import com.nasmedia.admixerssp.compose.rememberRewardVideoAd
import com.nasmedia.admixerssp.common.core.AdNetworkType

@Composable
fun RewardScreen() {
    val rewardListener = remember {
        object : AdListener() {
            override fun onReceivedAd(networkType: AdNetworkType, ad: Any) {}
            override fun onFailedToReceiveAd(code: Int, msg: String?) {}
            override fun onAdRewarded() { /* 보상 지급 */ }
        }
    }
    val reward = rememberRewardVideoAd(
        adUnitId = MyApplication.ADUNIT_ID_REWARD_VIDEO,
        listener = rewardListener,
    )

    LaunchedEffect(Unit) { reward.load() }
    Button(onClick = { if (reward.isLoaded) reward.show() }) { Text("리워드 보기") }
}
```

- API는 전면과 동일(`load()`/`show()`/`cancelLoad()`/`isLoaded`). 보상 지급은 `listener.onAdRewarded()`로 통지됩니다.
  Compose 헬퍼는 전용 보상 리스너 없이 노출하므로, 보상은 `AdListener` 경로로 옵니다.
- **지급 이력 키가 필요하면 `RewardInfo` 오버로드를 구현하세요.** 지급 1건당 고유 `transaction_id`가 전달되며, 서버 포스트백과 대조할 수 있습니다.

  ```kotlin
  object : AdListener() {
      override fun onAdRewarded(info: RewardInfo) {
          giveReward(info.transactionId)   // 이 오버로드를 구현하면 무인자 버전은 호출되지 않습니다
      }
  }
  ```

- **지급 중복 방지(dedup)는 앱 책임입니다.** 노출 1회당 지급 1회를 앱에서 보장하세요(one-shot 가드 등).
- ⚠️ **보상과 닫힘 콜백의 도착 순서는 네트워크 정책에 따라 달라질 수 있습니다.** 순서를 전제로 지급·알림 로직을 작성하지 말고 플래그로 판정하세요. 자세한 패턴은 [리워드 동영상 가이드의 보상 지급 안전 수칙](rewarded-video.md#보상-지급-안전-수칙)을 참고하세요.

---

## 전면 비디오 — `rememberVideoInterstitialAd`

보상이 없는 **전면 비디오 광고**(`AMMVideoInterstitial`)도 전면·리워드와 동일한 state-holder 패턴으로 제어합니다. 헬퍼가 매니저를 컴포지션당 1개만 생성하고, 이탈 시 `stop()`으로 정식 해제합니다.

```kotlin
import com.nasmedia.admixerssp.compose.rememberVideoInterstitialAd

@Composable
fun VideoScreen() {
    val video = rememberVideoInterstitialAd(
        adUnitId = MyApplication.ADUNIT_ID_VIDEO,
        // autoShow = false (기본): 수신 후 원하는 시점에 show() 직접 호출
    )

    LaunchedEffect(Unit) { video.load() }
    Button(onClick = { if (video.isLoaded) video.show() }) { Text("전면 비디오 보기") }
}
```

- API는 전면과 동일(`load()`/`show()`/`cancelLoad()`/`isLoaded`). 내부적으로 정식 메서드 `loadAd()`/`showAd()`/`stop()`를 호출합니다(구 `loadInterstitialVideoAd`/`showInterstitialVideoAd`는 deprecated).

> 🚨 전면과 동일하게 **수신 콜백에서 로드+자동노출을 재호출하지 마세요**(무한 재로드). 노출은 항상 `show()`로 하며, 헬퍼가 이 규약을 강제합니다.

---

## 콜백 매트릭스

`admixer-compose`의 모든 헬퍼는 **단일 `AdListener`** 로 이벤트를 전달합니다. View API에서 전면류가 쓰던 `FullScreenContentCallback` 대신, Compose에서는 배너·네이티브·전면·전면비디오·리워드가 모두 `AdListener` 한 인터페이스로 통일됩니다. (헬퍼가 `destroyed` 가드로 dispose 이후 도착하는 콜백은 자동 무시)

포맷별로 실제 발화하는 콜백은 다음과 같습니다.

| `AdListener` 콜백 | 배너 | 네이티브 | 전면 | 전면비디오 | 리워드 |
|---|:---:|:---:|:---:|:---:|:---:|
| `onReceivedAd` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `onFailedToReceiveAd` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `onAdShowFailed` | – | – | ✅ | ✅ | ✅ |
| `onAdDisplayed` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `onAdClicked` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `onAdClosed` | – | – | ✅ | ✅ | ✅ |
| `onAdCompleted` | – | – | – | ✅ | ✅ |
| `onAdSkipped` | – | – | – | ✅ | ✅ |
| `onAdRewarded` | – | – | – | – | ✅ |

> ⚠️ **`onAdCompleted`·`onAdSkipped` 는 네트워크에 따라 발화하지 않을 수 있습니다.** 재생 완료·스킵을 별도 신호로 통지할지는 각 네트워크 SDK가 결정하며, SDK는 이를 합성하지 않습니다. **보상 지급 판정은 반드시 `onAdRewarded` 로만** 하세요.
>
> 위 표는 해당 포맷에서 그 콜백이 **전달될 수 있는지**를 나타냅니다. 매번 발화한다는 뜻이 아닙니다.

> ✅ **`onFailedToReceiveAd`는 표준 콜백 `(errorCode, errorMsg)` 하나만 구현하면 됩니다.** 전 네트워크 No-Ad(`"Mediation"`)·SDK 미초기화(`"SDK"`) 등 내부 실패를 포함한 모든 수신 실패가 이 콜백 하나로 옵니다. 기존 4-인자 오버로드(String/enum)는 둘 다 `@Deprecated`(3.0 제거 예정)이며, 기본 구현이 표준 콜백으로 위임하므로 기존 코드도 동작은 그대로입니다.
>
> ```kotlin
> val listener = remember {
>     object : AdListener() {
>         override fun onFailedToReceiveAd(code: Int, msg: String?) {
>             isLoading = false // 수신 실패(최종) — 로딩 해제 등 최종 처리
>         }
>     }
> }
> ```

- `–` 는 해당 포맷에서 발생하지 않는 이벤트입니다(예: 배너/네이티브는 닫힘·재생완료·스킵 이벤트 없음).
- `onAdShowFailed` 는 **로드 성공 후 표시(show) 단계 실패**로, 인라인(배너/네이티브)에는 해당되지 않습니다.
- 배너/네이티브는 `listener` 파라미터로, 전면류는 `rememberInterstitialAd`/`rememberVideoInterstitialAd`/`rememberRewardVideoAd`의 `listener` 인자로 위 콜백을 받습니다. 보상 적립은 `onAdRewarded`로 통지됩니다.

---

## 헬퍼 없이 직접 호스팅하는 경우

`admixer-compose`를 쓰지 않고 직접 연동한다면, **아래 두 가지를 반드시** 구현해야 합니다.

### 1) 해제: `DisposableEffect.onDispose`에서 `stop()`

```kotlin
val context = LocalContext.current   // androidx.compose.ui.platform.LocalContext
val view = remember { AMMBannerView(context).apply { setAdInfo(AdInfo.Builder(adUnitId).build()); loadAd() } }
AndroidView(factory = { view })
DisposableEffect(adUnitId) { onDispose { view.stop() } }
```

### 2) 생명주기 연결 (필수)

> ⚠️ Compose는 Activity의 `onResume/onPause`를 자식 View로 **자동 전달하지 않습니다.** 아래를 연결하지 않으면 **백그라운드에서도 갱신/재생이 계속됩니다.**

```kotlin
val owner = LocalLifecycleOwner.current
DisposableEffect(owner, adUnitId) {
    val obs = LifecycleEventObserver { _, e ->
        when (e) {
            Lifecycle.Event.ON_RESUME -> view.onResume()
            Lifecycle.Event.ON_PAUSE  -> view.onPause()
            else -> {}
        }
    }
    owner.lifecycle.addObserver(obs)
    onDispose { owner.lifecycle.removeObserver(obs) }
}
```

---

## LazyColumn / LazyRow 내 광고

> ⚠️ 리스트 항목이 화면 밖으로 스크롤되면 dispose되고, 복귀 시 재구성됩니다. 항목을 **안정적인 key로 고정**하지 않으면 슬롯 재사용으로 상태가 꼬여 **광고가 사라져 돌아오지 않을** 수 있습니다.

```kotlin
LazyColumn {
    items(items, key = { it.id }) { item ->   // ← 안정적 key 필수
        if (item.isAd) AdMixerBanner(adUnitId = item.adUnitId)
        else ContentRow(item)
    }
}
```
- 광고 상태(로드/실패)는 항목 key에 묶어 hoisting하거나, `AdMixerBanner`처럼 `remember(adUnitId)`로 관리하세요.

---

## 문의

Compose 연동 문의는 **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)** 로 연락해 주세요.
