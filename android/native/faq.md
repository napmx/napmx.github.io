# 자주 묻는 질문 (FAQ)

> 광고가 안 나오거나 빌드가 안 될 때는 먼저 [광고가 노출되지 않을 때](#광고가-노출되지-않을-때-점검-순서)와 [빌드 / Gradle](#빌드--gradle) 섹션을 확인하세요. 실전 연동 문의는 [Q&A](qna.md), 에러 코드는 [에러 코드](error-codes.md)를 함께 참고하세요.

---

## 초기화 및 설정

**Q. 하나의 앱에 여러 개의 Media Key를 적용할 수 있나요?**

한 개의 앱에는 **한 개의 Media Key만** 적용 가능합니다. 여러 키를 번갈아 사용하면 광고가 정상 동작하지 않습니다.

---

**Q. 동일한 AdUnit ID로 여러 광고 객체를 생성해도 되나요?**

안 됩니다. 한 개의 AdUnit ID는 **한 개의 광고 객체에서만** 사용하세요. 동일한 AdUnit ID를 여러 객체에서 동시에 사용하면 광고가 정상적으로 동작하지 않습니다.

---

**Q. SDK 초기화는 언제 어디서 해야 하나요?**

`Application.onCreate()`에서 광고 로드 전에 **1회만** 호출하세요.

```java
@Override
public void onCreate() {
    super.onCreate();
    AdMixer.getInstance().initialize(this, MEDIA_KEY, adUnits);
}
```

`initialize()`를 호출하지 않거나 광고 로드보다 늦게 호출하면 `AX_ERR_INIT`(초기화 오류)가 발생합니다.

---

**Q. 각 네트워크 SDK를 `Application`에서 직접 초기화해야 하나요? (`PAGSdk.init`, `MobileAds.initialize` 등)**

대부분 **불필요**합니다. 워터폴에서 각 네트워크 SDK를 어댑터가 **요청 시 자동(lazy) 초기화**합니다. `Application`에서 `PAGSdk.init(...)`, `MobileAds.initialize()` 같은 호출은 제거해도 됩니다. 단, Google AdManager의 `com.google.android.gms.ads.APPLICATION_ID` 매니페스트 meta-data는 **반드시 유지**해야 합니다(아래 네트워크별 참고).

---

**Q. 지원하는 최소 OS 버전은?**

Android 5.0(API 21, Lollipop) 이상입니다. 단, 일부 네트워크 SDK(GMS 의존 등)는 더 높은 minSdk를 요구할 수 있어, 빌드 시 `manifest merger`가 호스트 앱 minSdk를 올리도록 요구할 수 있습니다.

---

**Q. 상세 로그를 확인하려면?**

초기화 전에 로그 레벨을 `VERBOSE`로 설정하고 LogCat에서 `AdMixerSDK` 접두사로 필터링하세요.

```java
AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE);
```
```
adb logcat | grep AdMixerSDK
```
> ⚠️ LogCat 태그는 `AdMixerSDK::<버전>` 형태(하위 태그가 붙으면 `AdMixerSDK::<버전>::<컴포넌트>`)로 **버전이 함께 붙습니다.** `adb logcat -s`는 태그 **완전 일치** 필터라 이 태그에는 사용할 수 없으니 위처럼 접두사로 필터링하세요.

운영 빌드에서는 `ERROR` 또는 `NONE`을 권장합니다.

---

## 빌드 / Gradle

**Q. `Could not resolve ...`(또는 `Failed to resolve`) 빌드 오류가 납니다.**

일부 네트워크는 **별도 Maven 저장소** 추가가 필요합니다. `settings.gradle`의 `dependencyResolutionManagement`(또는 프로젝트 `build.gradle`)에 추가하세요.

| 네트워크 | 추가 저장소 |
|---------|------------|
| Kakao Adfit | `https://devrepo.kakao.com/nexus/content/groups/public/` |
| Pangle | `https://artifact.bytedance.com/repository/pangle/` |
| Teads | `https://sdk.teads.tv/android/repo`, `https://teads.jfrog.io/artifactory/SDKAndroid-maven-prod` |

Google AdManager·AppLovin·Unity·NaverAdManager는 `google()` / `mavenCentral()` 만으로 해결됩니다. 자세한 설정은 [SDK 시작하기](getting-started.md#1-3-네트워크별-추가-maven-저장소)를 참고하세요.

---

**Q. 이미 다른 솔루션으로 같은 네트워크 SDK(예: play-services-ads)를 쓰고 있어 클래스 중복(`Duplicate class`) 오류가 납니다.**

해당 어댑터 의존성에서 중복 모듈을 `exclude` 하세요.

```gradle
implementation("io.github.nasmedia-tech:admixer-admanager:2.0.4") {
    exclude group: "com.google.android.gms", module: "play-services-ads"
}
```
exclude 후 Gradle 의존성 트리에서 동일 네트워크 SDK가 1개만 남는지, nap mx 광고와 기존 광고가 모두 정상 동작하는지 확인하세요. 자세한 내용은 [네트워크 SDK 중복 예외 처리](getting-started.md#네트워크-sdk-중복-예외-처리)를 참고하세요.

---

## 광고가 노출되지 않을 때 (점검 순서)

**Q. 광고가 전혀 노출되지 않습니다. 무엇부터 확인하나요?**

아래 순서로 점검하세요.

1. `AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE)` 설정 후 LogCat(`adb logcat | grep AdMixerSDK`)에서 워터폴 로그 확인
2. `AdMixer.getInstance().initialize(...)`가 광고 로드 **전에** 호출되었는지, Media Key가 올바른지 확인
3. AdUnit ID가 파트너 사이트에 등록된 값과 일치하는지 확인
4. 파트너 사이트에서 해당 AdUnit의 광고 설정이 **활성화**되어 있는지 확인
5. 인라인(배너/네이티브/인라인 동영상)은 `loadAd()` 후 **`addView`로 화면에 부착**했는지, 전면형은 수신 콜백 뒤 **`show(activity)`** 를 호출했는지 확인
6. 네트워크 연결 상태 확인

콜백 `onFailedToReceiveAd(...)`의 `errorCode`로 원인을 좁힐 수 있습니다([에러 코드](error-codes.md)).

---

**Q. `onFailedToReceiveAd`로 재고 없음(no-fill)이 옵니다. 연동 문제인가요?**

`AX_ERR_NO_ADS`(재고 부족)는 대개 **연동 오류가 아니라 광고 재고/설정** 이슈입니다. 워터폴의 모든 네트워크가 광고를 채우지 못하면 이 코드 하나로 통지됩니다.

> 🚫 **`AX_ERR_NO_FILL`로 분기하지 마세요.** 상수는 존재하지만 SDK가 전달하지 않습니다 — no-fill은 **전부 `AX_ERR_NO_ADS`** 로 옵니다([에러 코드](error-codes.md)). 일정 시간 후 재요청하고, 지속되면 해당 AdUnit·네트워크 설정(운영 Ad Unit 프로비저닝 포함)을 운영팀에 확인하세요. **테스트 Ad Unit**은 노필/서버 에러가 정상일 수 있어 **운영 Ad Unit으로 전환**이 필요합니다.

---

**Q. 특정 네트워크 광고만 나오지 않습니다.**

1. `build.gradle`에 해당 어댑터 모듈(`admixer-xxx`) 의존성이 있는지 확인(없으면 `AX_ERR_NO_ADAPTER`)
2. 워터폴 로그에 `[SKIP] ... (Missing Keys)`가 있으면 서버에 네트워크 필수 키(예: Pangle `placement_id`, AppLovin `zone_id`)가 없는 경우입니다 → 운영팀에 키 프로비저닝 요청 또는 `setAdapterConfig`로 주입([Q&A](qna.md) 참고)
3. Google AdManager: 매니페스트 `APPLICATION_ID` 확인
4. NaverAdManager: 운영 Ad Unit ID 발급 여부 확인(`PUBLISHER_CD`는 SDK 제공)
5. Adfit: **Activity Context**로 뷰를 생성했는지 확인

---

## 배너 광고

**Q. `AMMBannerView`를 만들었는데 광고가 표시되지 않습니다.**

광고가 표시되려면 반드시 `container.addView(adView)`로 레이아웃에 부착해야 합니다. 뷰가 화면에 부착되는 시점에 **자동 노출**되므로 `showAd()` 호출은 필요 없습니다.

- **즉시 노출**: `loadAd()` 호출 시점에 이미 부착돼 있으면 수신 후 자동 표시
- **지연 노출**: `loadAd()`로 미리 로드한 뒤, 원하는 시점에 `addView(adView)`로 부착하면 그때 노출

---

**Q. AdUnit 설정 사이즈와 다른 크기로 노출됩니다.**

광고 네트워크·소재 유형에 따라 실제 노출 사이즈가 달라질 수 있어 설정 사이즈가 보장되지 않습니다. 컨테이너를 **너비 `match_parent` + 높이 `wrap_content`** 로 두면 소재 크기에 맞게 조정됩니다. 특히 **AdManager 표준 배너는 디바이스 너비 기반 anchored adaptive**로 요청되어 높이가 달라질 수 있으며, MREC(300×250) 등 고정 슬롯은 종전대로 유지됩니다.

---

**Q. 배너가 잘리거나 위아래 여백이 생깁니다.**

배너 컨테이너 높이를 고정하지 마세요. `wrap_content`를 사용하면 소재 높이에 맞춰 표시됩니다.

---

## 네이티브 광고

**Q. `setViewBinder()`를 설정하지 않으면 어떻게 되나요?**

네이티브 광고가 **렌더링되지 않습니다.** `loadAd()` 호출 전에 반드시 `setViewBinder(NativeAdViewBinder)`를 설정하세요.

---

**Q. 레이아웃에 `RelativeLayout`을 꼭 써야 하나요?**

`RelativeLayout` 사용을 강력히 권장합니다. 다른 레이아웃이 필요하면 `RelativeLayout`으로 감싸고 내부에 배치하세요.

---

**Q. 제목/설명 등 일부 asset이 빈 칸으로 표시되거나, 반대로 표시되지 않습니다.**

서버 소재에 선택(optional) asset이 없으면 SDK/어댑터가 해당 View를 빈 칸이 아니라 **자동으로 `GONE` 처리**합니다. 모든 asset View를 선언해 두어도 소재에 없는 항목은 표시되지 않으니, 누락 케이스에서도 레이아웃이 자연스럽도록 상대 위치 제약(`layout_below` 등)으로 배치하세요. `title`/`icon`/`mainView` 중 최소 1개는 반드시 사용해야 합니다.

---

**Q. Naver 통합형 지면에서 레이아웃 바인딩이 적용되지 않습니다.**

Naver Native Simple(템플릿형) 응답은 NAM SDK가 **소재 전체를 템플릿으로 렌더링**하므로 `NativeAdViewBinder`의 자산 매핑이 적용되지 않으며, 높이는 소재 비율로 자동 결정됩니다. 연동 방법은 일반 네이티브와 동일하고 응답 타입 분기는 SDK가 자동 처리합니다.

---

## 전면 / 리워드 / 동영상

**Q. 로드와 동시에 자동 노출하던 `startInterstitial()` 같은 메서드는 어디 갔나요?**

즉시 노출(로드+자동 노출) API는 **v2.0.0에서 제거**되었습니다(`startInterstitial()`/`startInterstitialVideoAd()`/`startRewardVideoAd()`). 모든 광고는 **로드(수신) → 노출**이 분리됩니다: 전면형은 정적 `loadAd()` 수신 콜백 뒤 원하는 시점에 `show(activity)`, 인라인은 `addView` 부착 시점에 자동 노출됩니다. 수신 즉시 노출하려면 수신 콜백 안에서 `show(activity)`를 바로 호출하세요.

---

**Q. 전면 광고를 `show(activity)` 할 때 Activity가 꼭 필요한가요?**

네. **노출 시점에는 Activity Context가 필요**합니다(`loadAd()`는 Application Context로도 가능). Application Context로 `show()`를 호출하면 표시되지 않습니다.

---

**Q. 표시 중인 광고는 유지하면서 진행 중 로드만 취소하려면?**

`cancelLoad()`를 호출하세요. 로딩 중일 때만 취소하고 표시 중(SHOWING)이면 아무 동작도 하지 않습니다. 전체 정리(리스너 해제 포함)는 `stop()`입니다.

---

**Q. EARNEDREWARD와 COMPLETE의 차이는? Skip하면 리워드가 지급되나요?**

- `onAdRewarded()`(EARNEDREWARD): 리워드 지급 조건 충족(시청 완료). **리워드 지급은 이 시점**에서 처리하세요. (정적 `loadAd` 방식은 `OnUserEarnedRewardListener.onUserEarnedReward()`)
- `onAdCompleted()`(COMPLETE): 동영상 재생이 끝까지 완료. 네트워크에 따라 EARNEDREWARD와 동시 또는 별도로 발생
- `onAdSkipped()`(SKIPPED): **리워드 미지급** 상황. 리워드는 반드시 EARNEDREWARD에서만 지급하세요.

---

**Q. 리워드를 서버에서 안전하게 검증하고 싶습니다.**

S2S Reward Callback을 사용하세요. 파트너 사이트에 콜백 URL을 등록하고, 필요 시 `AdInfo.Builder.setCustomParams(...)`로 사용자 식별자 등을 전달합니다. 자세한 내용은 [리워드 동영상 — S2S Reward Callback](rewarded-video.md#s2s-reward-callback-서버-간-리워드-검증)을 참고하세요.

---

**Q. 배너·네이티브 광고는 자동으로 갱신되나요?**

서버 광고 단위 `interval`(초)이 **0보다 클 때만** 노출 후 `interval`마다 자동 갱신되고, 실패 시 재요청합니다(최소 5초 간격). `interval`이 0(미설정)이면 단발성(자동 재로드 없음)입니다. 무한 실패-재로드는 SDK 내부 가드가 차단합니다. *(v2.0.0: 기존 `isRetry`/`maxRetryCountInSlot` 옵션은 제거되고 서버 `interval`로 일원화)*

---

## 미디에이션 / 네트워크별

**Q. Adfit 상용 광고는 언제부터 응답되나요?**

Adfit은 매체 심사 후 상용 광고가 응답됩니다.

```
연동 테스트 → 매체 라이브 → Adfit 매체 심사 신청 → 심사 완료 → 상용 광고 송출
```
심사 신청은 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요.

---

**Q. Adfit 광고가 응답은 되는데 표시가 안 됩니다.**

Adfit은 **Activity Context**가 필요합니다. `AMMBannerView`/`AMMNativeAdView`를 `getApplicationContext()`가 아니라 **Activity(`this`)** 로 생성하세요.

---

**Q. Google AdManager를 추가했더니 앱이 시작하자마자 크래시 납니다.**

`AndroidManifest.xml`에 `com.google.android.gms.ads.APPLICATION_ID` meta-data가 없으면 GMA SDK가 초기화 중 크래시합니다. 발급받은 App ID를 추가하세요.

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="발급받은 Google App ID" />
```
App ID 발급은 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의하세요.

---

**Q. Google AdManager를 이미 운영 중인 앱에 추가해도 되나요?**

기존 운영 지면과 **다른 지면**에 한해 중복 사용이 가능합니다. 동일 지면이면 `exclude`로 중복을 방지하세요([중복 예외 처리](getting-started.md#네트워크-sdk-중복-예외-처리)).

---

**Q. Pangle은 `app_id`/`placement_id`를 어디에 설정하나요?**

Pangle SDK init은 어댑터가 자동 처리합니다. `app_id`/`placement_id`는 서버(media-conf)에서 전달되며, 서버에 없으면 `AdInfo.Builder.setAdapterConfig(AdMixer.ADAPTER_PANGLE, {...})`로 주입할 수 있습니다([Q&A](qna.md)).

---

**Q. AppLovin SDK Key는 어디에 넣나요?**

번들 AppLovin은 코드 기반 초기화이므로 `setAdapterConfig(AdMixer.ADAPTER_APPLOVIN, {"sdkKey": "..."})`로 전달합니다(미지정 시 어댑터 기본 키 사용). 매니페스트 `applovin.sdk.key`는 사용하지 않습니다.

---

**Q. Naver Ad Manager의 `PUBLISHER_CD`를 매니페스트에 넣어야 하나요?**

**아니요.** `com.naver.gfpsdk.PUBLISHER_CD`는 SDK(`admixer-naveradmanager` aar)가 제공·관리하므로 호스트 앱에 설정하지 마세요(이미 추가했다면 제거 권장).

---

## ProGuard / R8

**Q. 디버그에서는 정상인데 릴리즈(난독화) 빌드에서 광고가 안 나오거나 크래시 납니다.**

**앱에 별도 keep 규칙을 추가할 필요는 없습니다.** Core SDK와 각 어댑터 AAR이 `consumer-rules.pro`를 내장하고 있어, Gradle이 의존성을 해석할 때 호스트 앱의 릴리즈 빌드(R8/ProGuard)에 **자동으로 병합**됩니다.

그래도 릴리즈 빌드에서만 문제가 생긴다면 아래를 확인하세요.

1. 사용 중인 SDK·어댑터 버전이 최신인지 확인
2. `build/outputs/mapping/release/seeds.txt`에 `nasmedia` 클래스가 보호 목록으로 남아 있는지 확인 — 없으면 consumer-rules가 병합되지 않은 것입니다
3. 앱이 SDK 클래스를 **리플렉션으로 직접 참조**한다면 해당 클래스만 개별 keep
4. 해당 네트워크의 공식 ProGuard 가이드 확인

자세한 내용과 확인 방법은 [ProGuard 설정](proguard.md)을 참고하세요.

---

## 개인정보 및 테스트

**Q. 아동 대상 앱인데 뭘 설정해야 하나요?**

`AdMixer.setTagForChildDirectedTreatment(AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE)`를 `Application.onCreate()`에서 설정하세요. Google Play **Families 정책**은 국가와 무관하게 적용되므로 국내 서비스도 필수입니다. 자세한 내용은 [개인정보 / 테스트 설정](privacy.md)을 참고하세요.

---

**Q. GDPR/CCPA 동의는 어떻게 전달하나요?**

국내(한국) 서비스는 GDPR(EU)·CCPA(캘리포니아) 적용 대상이 아니라 별도 설정이 필요하지 않습니다. **해외 서비스로 확장하거나 EU·미국 트래픽이 있다면** 네트워크마다 지원 범위가 달라 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의해 주세요. 사용 중인 네트워크 조합에 맞춰 안내해 드립니다.

---

**Q. 테스트 광고는 어떻게 받나요? 테스트 기기는 어떻게 지정하나요?**

초기화 시 `AdMixer.setTestMode(true)`와 `AdMixer.setTestDeviceIds([GAID])`를 설정하세요. 기기의 **광고 ID(GAID)** 는 안드로이드 설정 → Google → 광고에서 확인하거나, 초기화 후 LogCat 로그에서도 확인할 수 있습니다. AppLovin·Unity·AdManager·Pangle에 반영됩니다([privacy.md](privacy.md)).

---

## 비즈보드

**Q. 카카오 비즈보드를 연동하려면?**

비즈보드는 별도 코드 발급이 필요합니다. 발급·심사 및 연동 방법 안내는 [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)로 문의해 주세요.

---

## 문의

해결되지 않는 문의는 **[nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)** 로 연락해 주세요. `AdMixerLog.setLogLevel(AdMixerLog.LogLevel.VERBOSE)` 설정 후 `adb logcat | grep AdMixerSDK`로 수집한 로그를 함께 전달하면 진단이 빠릅니다.
