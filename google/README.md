# Google 수익화 연동

nap mx를 통해 Google 광고 수익화를 진행하는 방법입니다.

> Google 광고 수익화는 Google의 게시자 정책을 반드시 준수해야 합니다.

문의: [nap_mx@nasmedia.co.kr](mailto:nap_mx@nasmedia.co.kr)

---

## 시작하기 전에

### 앱 요구사항

- 주요 앱 스토어(Google Play, Apple App Store 등) 중 **최소 1개** 등록 필수
- `app-ads.txt` 파일 업로드 필수

### 웹사이트 요구사항

- **자체 콘텐츠**를 보유한 사이트여야 합니다
- `ads.txt` 파일 업로드 필수

### Google 정책 확인

Google 광고 연동 전 반드시 아래 정책을 확인하세요.

- [Google 게시자 정책](https://support.google.com/adsense/answer/48182)
- [Google AdSense 프로그램 정책](https://support.google.com/adsense/answer/9335564)
- [보상형 광고 정책](https://support.google.com/admob/answer/9027537)

---

## MCM 연동 절차

**MCM(Multiple Customer Management)**은 Google Ad Manager 계정을 nap mx와 연결하는 방식입니다.

| 단계 | 내용 | 소요 기간 |
|------|------|----------|
| 1 | Google Ad Manager 계정 확인 및 신청 | - |
| 2 | nap mx로부터 MCM 초대장 수락 | 즉시 |
| 3 | `sellers.json` 업데이트 | 즉시 |
| 4 | `ads.txt` / `app-ads.txt` 등록 | 즉시 |
| 5 | 앱/사이트 승인 | **3~4 영업일** |

---

## ads.txt / app-ads.txt 등록

Google 인증을 위해 다음 내용을 `ads.txt`(웹) 또는 `app-ads.txt`(앱)에 추가합니다.

```
# nap mx (nasmedia)
admixer.co.kr, [발급받은_PUBLISHER_ID], DIRECT, [TAG_ID]
googlesyndication.com, pub-XXXXXXXXXXXXXXXX, RESELLER, f08c47fec0942fa0
```

> 실제 Publisher ID와 TAG ID는 파트너 사이트에서 확인하거나 담당자에게 문의하세요.

**앱의 경우** — 앱 개발사 공식 웹사이트에 `app-ads.txt` 파일을 호스팅해야 합니다.

```
https://your-company.com/app-ads.txt
```

---

## nap mx 파트너 사이트에서 Google 수익화 활성화

1. [파트너 사이트](https://publisher.admixer.co.kr/) 로그인
2. **미디어 > 미디어 관리** → 대상 미디어 선택
3. **네트워크 설정** → `Google AdManager` 활성화
4. 발급받은 **Google Ad Manager 네트워크 코드** 입력
5. **저장**

---

## Hybrid App (WebView) 추가 설정

WebView 환경에서 Google 광고가 동작하려면 ADID 전송과 Google WebView API 적용이 필요합니다.

### Android

```java
// Google WebView API 활성화
import com.google.android.gms.ads.MobileAds;

MobileAds.initialize(this, initializationStatus -> {});

// WebView에 ADID 전송
import com.google.android.gms.ads.identifier.AdvertisingIdClient;

new Thread(() -> {
    try {
        AdvertisingIdClient.Info adInfo =
            AdvertisingIdClient.getAdvertisingIdInfo(context);
        String adId = adInfo.getId();
        // adId를 WebView JavaScript로 전달
        webView.evaluateJavascript(
            "window.GAID = '" + adId + "';", null);
    } catch (Exception e) { e.printStackTrace(); }
}).start();
```

### iOS

```swift
import AppTrackingTransparency
import AdSupport

ATTrackingManager.requestTrackingAuthorization { status in
    let idfa = ASIdentifierManager.shared().advertisingIdentifier.uuidString
    DispatchQueue.main.async {
        self.webView.evaluateJavaScript(
            "window.IDFA = '\(idfa)';", completionHandler: nil)
    }
}
```
