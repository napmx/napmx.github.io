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
- 컴포지션 진입 시 `AdView` 생성 + `loadAd()`, 이탈 시 `DisposableEffect.onDispose`에서 `destroy()`.
- `ON_RESUME/ON_PAUSE`를 `adView.onResume()/onPause()`로 자동 연결 → **백그라운드 갱신/영상 재생 정지**.
- dispose 이후 도착하는 콜백은 무시(가드) → 스크롤 아웃/슬롯 재사용 시 상태 오염 방지.

리스너가 필요하면:

```kotlin
val listener = remember {
    object : AdListener {
        override fun onReceivedAd(adapterName: String, ad: Any) {}
        override fun onFailedToReceiveAd(ad: Any?, adapterName: String, code: Int, msg: String?) {}
        override fun onEventAd(ad: Any, event: AdEvent) {}
    }
}
AdMixerBanner(adUnitId = "...", listener = listener)
```

---

## 헬퍼 없이 직접 호스팅하는 경우

`admixer-compose`를 쓰지 않고 직접 연동한다면, **아래 두 가지를 반드시** 구현해야 합니다.

### 1) 해제: `DisposableEffect.onDispose`에서 `destroy()`

```kotlin
val view = remember { AdView(context).apply { setAdInfo(AdInfo.Builder(adUnitId).build()); loadAd() } }
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

## 전면 / 리워드 (안내)

전면·리워드는 **Activity 기반 노출**이라 별도 상태 홀더로 제공될 예정입니다(다음 릴리즈). 현재는 `InterstitialAd`/`RewardInterstitialVideoAd`를 Activity Context로 사용하되, 다음을 지키세요.

> 🚨 **수신 콜백(`onReceivedAd`)에서 `startInterstitial()`(=로드+자동노출)을 호출하지 마세요.** 매 수신마다 재로드되어 무한 루프가 발생합니다. 표시는 **`showInterstitial()`** 을 사용하세요. (자동 노출은 `startInterstitial()`을 최초 1회만)

---

## 문의

Compose 연동 문의는 **nap_mx@nasmedia.co.kr** 로 연락해 주세요.
