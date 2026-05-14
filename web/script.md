# WEB Script 연동

nap mx Script를 웹 페이지에 삽입하여 광고를 노출하는 방법입니다.  
M.Web, PC.Web 환경에서 사용합니다.

---

## 사전 준비

[파트너 사이트](https://publisher.admixer.co.kr/)에서 **Media Key**와 **Adunit ID**를 발급받아야 합니다.

---

## 1. 기본 Script 삽입

`<head>` 또는 `<body>` 상단에 nap mx 공통 스크립트를 삽입합니다.

```html
<script src="https://cdn.admixer.co.kr/napmx.js" async></script>
```

---

## 2. 배너 광고 삽입

광고를 노출할 위치에 컨테이너 `<div>`를 추가하고 스크립트로 광고를 요청합니다.

```html
<!-- 광고 컨테이너 -->
<div id="napmx-banner-1"></div>

<script>
  napmx.cmd.push(function() {
    napmx.defineSlot('발급받은_ADUNIT_ID', [320, 50], 'napmx-banner-1')
         .addService(napmx.pubads());
    napmx.pubads().enableSingleRequest();
    napmx.enableServices();
  });
</script>
```

---

## 3. 다중 광고 슬롯

한 페이지에서 여러 광고를 동시에 요청하려면 `enableSingleRequest()`를 사용합니다.

```html
<!-- 상단 배너 -->
<div id="napmx-top"></div>
<!-- 하단 배너 -->
<div id="napmx-bottom"></div>

<script>
  napmx.cmd.push(function() {
    napmx.defineSlot('ADUNIT_ID_TOP', [320, 50], 'napmx-top')
         .addService(napmx.pubads());
    napmx.defineSlot('ADUNIT_ID_BOTTOM', [320, 100], 'napmx-bottom')
         .addService(napmx.pubads());

    napmx.pubads().enableSingleRequest();
    napmx.enableServices();
  });
</script>
```

---

## 4. 지원 광고 사이즈

| 포맷 | 사이즈 | 플랫폼 |
|------|--------|--------|
| 배너 (소형) | 320×50 | M.Web |
| 배너 (중형) | 320×100 | M.Web |
| 배너 (사각) | 300×250 | M.Web / PC.Web |
| 배너 (전면) | 320×480 | M.Web |
| 배너 (세로) | 120×600 | PC.Web |
| 네이티브 | 자유 | M.Web / PC.Web |

---

## 5. 광고 이벤트 콜백

```html
<script>
  napmx.cmd.push(function() {
    var slot = napmx.defineSlot('발급받은_ADUNIT_ID', [320, 50], 'napmx-banner-1')
                    .addService(napmx.pubads());

    // 광고 수신 성공
    napmx.pubads().addEventListener('slotRenderEnded', function(event) {
      if (!event.isEmpty) {
        console.log('광고 수신:', event.slot.getAdUnitPath());
      } else {
        console.log('광고 없음 (No Fill)');
      }
    });

    napmx.pubads().enableSingleRequest();
    napmx.enableServices();
  });
</script>
```

---

## 주의사항

- `napmx.js` 스크립트는 페이지당 **1회만** 삽입합니다.
- `async` 속성을 사용하면 페이지 로드 성능에 영향을 주지 않습니다.
- 광고 컨테이너 `<div>`는 스크립트 실행 전에 DOM에 존재해야 합니다.
