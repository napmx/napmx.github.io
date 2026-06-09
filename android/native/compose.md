# Jetpack Compose 연동

Compose 앱에서 nap mx 광고를 연동하는 방법입니다. 코어 SDK는 View 기반이므로, Compose에서는 **생명주기 연결**과 **해제 시점**을 직접 관리해야 합니다. 이를 자동 처리하는 **공식 헬퍼 모듈 `admixer-compose`** 를 제공합니다.

> ℹ️ 기본 설정(미디어 키/초기화/ProGuard)은 [SDK 시작하기](getting-started.md)를 먼저 완료하세요.

---

## 의존성 추가

```gradle
dependencies {
    implementation 'io.github.nasmedia-tech:admixer-ssp:2.0.0'        // 코어(필수)
    implementation 'io.github.nasmedia-tech:admixer-compose:2.0.0'    // Compose 헬퍼
    // 사용하는 어댑터 모듈 + play-services-ads-identifier 등은 시작하기 가이드 참고
}
```

---

## 배너 — `AdMixerBanner`

`AndroidView` 호스팅·`destroy()` 해제·생명주기 연결·도착 콜백 가드를 헬퍼가 자동 처리합니다.

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
- 컴포지션 진입 시 `AMMBannerView` 생성 + `loadAd()`, 이탈 시 `DisposableEffect.onDispose`에서 `destroy()`.
- `ON_RESUME/ON_PAUSE`를 `adView.onResume()/onPause()`로 자동 연결 → **백그라운드 갱신/영상 재생 정지**.
- dispose 이후 도착하는 콜백은 무시(가드) → 스크롤 아웃/슬롯 재사용 시 상태 오염 방지.

리스너가 필요하면:

```kotlin
val listener = remember {
    object : AdListener() {
        override fun onReceivedAd(adapterName: String, ad: Any) {}
        override fun onFailedToReceiveAd(ad: Any?, adapterName: String, code: Int, msg: String?) {}
        override fun onAdDisplayed() {}
        override fun onAdClicked() {}
    }
}
AdMixerBanner(adUnitId = "...", listener = listener)
```

---

## 네이티브 — `AdMixerNativeAd`

네이티브 광고는 **레이아웃 바인더(`NativeAdViewBinder`)** 로 제목/이미지/CTA 등을 매핑합니다. 헬퍼가 `loadNativeAd()` → 수신 시 `showAd()`(어댑터 뷰 부착) → 이탈 시 `destroy()`까지 자동 처리합니다.

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

---

## 전면 — `rememberInterstitialAd`

전면·리워드는 **Activity 노출형**이라 화면에 배치하는 컴포저블이 아니라, `remember*`가 반환하는 **state-holder 핸들**로 제어합니다. 헬퍼가 매니저를 컴포지션당 1개만 생성하고(중복 콜백 방지, #58), 이탈 시 `stopInterstitial()`로 정식 해제합니다.

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

> 🚨 **수신 콜백에서 `start*`(로드+자동노출)를 직접 호출하지 마세요.** 매 수신마다 재로드되어 무한 루프가 발생합니다(#59). 노출은 항상 `show()`(=`showInterstitial()`)로 하며, 헬퍼는 이 규약을 강제합니다.

---

## 리워드 — `rememberRewardVideoAd`

```kotlin
import com.nasmedia.admixerssp.compose.rememberRewardVideoAd

@Composable
fun RewardScreen() {
    val rewardListener = remember {
        object : AdListener() {
            override fun onReceivedAd(adapterName: String, ad: Any) {}
            override fun onFailedToReceiveAd(ad: Any?, adapterName: String, code: Int, msg: String?) {}
            override fun onAdRewarded() { /* 보상 지급 */ }
        }
    }
    val reward = rememberRewardVideoAd(
        adUnitId = MyApplication.ADUNIT_ID_REWARD,
        listener = rewardListener,
    )

    LaunchedEffect(Unit) { reward.load() }
    Button(onClick = { if (reward.isLoaded) reward.show() }) { Text("리워드 보기") }
}
```

- API는 전면과 동일(`load()`/`show()`/`cancelLoad()`/`isLoaded`). 보상 지급은 `listener.onAdRewarded()`로 통지됩니다.

---

## 헬퍼 없이 직접 호스팅하는 경우

`admixer-compose`를 쓰지 않고 직접 연동한다면, **아래 두 가지를 반드시** 구현해야 합니다.

### 1) 해제: `DisposableEffect.onDispose`에서 `destroy()`

```kotlin
val view = remember { AMMBannerView(context).apply { setAdInfo(AdInfo.Builder(adUnitId).build()); loadAd() } }
AndroidView(factory = { view })
DisposableEffect(adUnitId) { onDispose { view.destroy() } }
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

Compose 연동 문의는 **nap_mx@nasmedia.co.kr** 로 연락해 주세요.
