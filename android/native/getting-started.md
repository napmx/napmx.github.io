# Android SDK 시작하기 (Native)

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. 권한 설정

`AndroidManifest.xml`에 인터넷 권한을 추가합니다.

```xml
<manifest>
    <uses-permission android:name="android.permission.INTERNET" />
    <application>
        ...
    </application>
</manifest>
```

Google AdManager 사용 시 추가 설정:

```xml
<application>
    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="발급받은 App ID" />
</application>
```

AppLovin 사용 시 추가 설정:

```xml
<application>
    <meta-data
        android:name="applovin.sdk.key"
        android:value="발급받은 키" />
</application>
```

---

## 2. Gradle 설정

프로젝트 수준 `build.gradle`에 Maven Central 저장소가 포함되어 있는지 확인합니다.

```groovy
// settings.gradle (또는 build.gradle)
repositories {
    google()
    mavenCentral()
}
```

앱 모듈 `build.gradle`에 SDK 의존성을 추가합니다.

```groovy
dependencies {
    // nap mx 메인 SDK (필수)
    implementation 'io.github.nasmedia-tech:admixer-ssp:1.0.23'

    // 광고 ID 수집 (필수)
    implementation 'com.google.android.gms:play-services-ads-identifier:18.2.0'

    // --- 선택적 어댑터 (사용하는 네트워크만 추가) ---

    // Google AdManager
    implementation 'com.google.android.gms:play-services-ads:23.x.x'

    // KakaoAdfit
    implementation 'com.kakao.adfit:ads-base:3.x.x'

    // AppLovin
    implementation 'com.applovin:applovin-sdk:12.x.x'

    // Pangle (ByteDance)
    implementation 'com.pangle.global:pag-sdk:7.x.x'
}
```

Pangle 사용 시 `settings.gradle`에 저장소를 추가합니다.

```groovy
maven { url "https://artifact.bytedance.com/repository/pangle/" }
```

KakaoAdfit 사용 시 저장소를 추가합니다.

```groovy
maven { url 'https://devrepo.kakao.com/nexus/content/groups/public/' }
```

---

## 3. SDK 초기화

`Application` 또는 첫 번째 `Activity`의 `onCreate()`에서 초기화합니다.

```java
// Java
import com.nasmedia.admixerssp.common.AdMixer;

public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        AdMixer.getInstance().init(this, "발급받은_MEDIA_KEY");
    }
}
```

```kotlin
// Kotlin
import com.nasmedia.admixerssp.common.AdMixer

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        AdMixer.getInstance().init(this, "발급받은_MEDIA_KEY")
    }
}
```

> **주의**: `init()`은 앱 실행 중 한 번만 호출해야 합니다.

---

## 4. ProGuard 설정

난독화를 사용하는 경우 아래 규칙을 `proguard-rules.pro`에 추가합니다.

```
-keep class com.nasmedia.admixerssp.** { *; }
-keep interface com.nasmedia.admixerssp.** { *; }
-dontwarn com.nasmedia.admixerssp.**
```

---

## 5. COPPA 설정 (선택)

아동 대상 앱의 경우 COPPA 플래그를 설정합니다.

```java
// 아동 대상 앱
AdMixer.tagForChildDirectedTreatment = AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE;

// 일반 앱
AdMixer.tagForChildDirectedTreatment = AdMixer.AX_TAG_FOR_CHILD_DIRECTED_TREATMENT_FALSE;
```

---

## 다음 단계

- [배너 광고 연동하기](/android/native/banner)
- [전면 광고 연동하기](/android/native/interstitial)
- [네이티브 광고 연동하기](/android/native/native-ad)
