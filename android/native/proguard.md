# ProGuard 설정

Core SDK(`admixer-ssp`)와 각 어댑터 모듈은 AAR에 `consumer-rules.pro`를 내장하고 있습니다. Gradle이 의존성을 해석할 때 이 규칙을 호스트 앱의 릴리즈 빌드(R8/ProGuard)에 **자동으로 병합**하므로, **앱의 `proguard-rules.pro`에 별도 규칙을 추가할 필요가 없습니다.**

---

## 자동 적용 여부

| 모듈 | 자동 적용 | 비고 |
|---------|----------|------|
| Core SDK (`admixer-ssp`) | ✅ | 공개 API(`AMM*` 등) · 미디에이션 SPI 전반 |
| Google AdManager | ✅ | - |
| Kakao Adfit | ✅ | - |
| Pangle | ✅ | - |
| AppLovin | ✅ | - |
| Unity Ads | ✅ | - |
| Naver Ad Manager | ✅ | - |
| Teads | ✅ | - |
| Interactive Labs (`admixer-interactive`) | ✅ | enum 이름·source `@Keep` 기반 최소 규칙 |
| 🧪 GMA NextGen **(beta)** | ✅ | AdManager·NaverAd와 공존 불가 |

빌드 경고나 난독화로 인한 런타임 오류(예: `ClassNotFoundException`, 리플렉션 실패)가 발생하면, 우선 사용 중인 SDK 버전이 최신인지 확인하고 해당 네트워크의 공식 ProGuard 가이드를 참고하세요.

> Kakao AdFit 3.21.17의 광고 식별자 INFO 로그 제거 규칙은 호스트 앱의 **minify가 활성화된 release 빌드**에서만 적용됩니다. AdFit 버전을 변경하면 내부 logger 바이트코드와 규칙을 다시 검증해야 하며, SDK 빌드는 검증되지 않은 버전으로 release/publish하는 것을 차단합니다.

---

## 참고 — 직접 keep이 필요한 경우

일반적인 연동에서는 위 자동 적용만으로 충분합니다. 다만 앱이 아래와 같이 **SDK 클래스를 리플렉션으로 직접 참조**하는 등 특수한 경우에는 해당 클래스만 개별적으로 keep하세요.

Interactive는 공개 패키지 전체를 keep하지 않습니다. 일반 Java/Kotlin 직접 호출은 R8 호출 그래프가
보존하며, 앱이 자체 `RewardedAdGateway`/정산 Handler를 reflection으로 생성할 때만 그 앱 구현을
개별 keep하세요. SDK 테스트용 Fake는 production AAR에 포함되지 않습니다.

```proguard
# 예시 — 앱 코드가 특정 어댑터 클래스를 리플렉션으로 직접 참조하는 경우에만 추가
-keep class com.nasmedia.pangle.PangleAdapter { public *; }
```

---

## 확인 방법

빌드 후 앱 모듈의 `build/outputs/mapping/release/` 폴더의 `seeds.txt`에서 보호된 클래스 목록을 확인할 수 있습니다.

```bash
# 빌드 후 mapping 파일 확인
cat app/build/outputs/mapping/release/seeds.txt | grep "nasmedia"
```
