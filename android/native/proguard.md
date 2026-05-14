# ProGuard 설정

릴리즈 빌드에서 ProGuard(R8) 난독화를 적용할 때 아래 규칙을 추가해야 합니다.

---

## 기본 설정

`proguard-rules.pro`에 아래 규칙을 추가하세요.

```proguard
##############################################
# nap mx AdMixer SDK — ProGuard Rules
##############################################

# ✅ 필수 — Core SDK
-keep class com.nasmedia.admixerssp.** { *; }

# 사용하는 어댑터 모듈만 추가하세요
-keep class com.nasmedia.admanager.** { *; }       # Google AdManager
-keep class com.nasmedia.adfit.** { *; }            # Kakao Adfit
-keep class com.nasmedia.pangle.** { *; }           # Pangle
-keep class com.nasmedia.applovin.** { *; }         # AppLovin
-keep class com.nasmedia.unity.** { *; }            # Unity Ads
-keep class com.nasmedia.mobwith.** { *; }          # Mobwith
-keep class com.nasmedia.naveradmanager.** { *; }   # Naver Ad Manager
-keep class com.nasmedia.teads.** { *; }            # Teads
```

---

## 네트워크 SDK 자체 규칙

각 네트워크 SDK 어댑터에는 자체 ProGuard 규칙이 포함되어 있으며, AAR 내 `consumer-rules.pro`로 자동 적용됩니다. 별도 추가 없이 동작하지만, 빌드 경고 발생 시 해당 네트워크의 공식 ProGuard 가이드를 참고하세요.

| 네트워크 | 자동 적용 |
|---------|----------|
| Google AdManager | ✅ |
| Kakao Adfit | ✅ |
| Pangle | ✅ |
| AppLovin | ✅ |
| Unity Ads | ✅ |
| Mobwith | ✅ |
| Naver Ad Manager | ✅ |
| Teads | ✅ |

---

## 확인 방법

빌드 후 `build/outputs/mapping/release/` 폴더의 `seeds.txt`에서 보호된 클래스 목록을 확인할 수 있습니다.

```bash
cat app/build/outputs/mapping/release/seeds.txt | grep "nasmedia"
```
