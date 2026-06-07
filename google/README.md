# Google MCM 연동

Google 광고 수익화를 위한 MCM(Multiple Customer Management) 연동 절차를 안내합니다.

---

## 1. Google 수익화 필수 조건

Google 광고 수익화를 진행하기 위해서는 아래 기준을 모두 충족해야 합니다.

<div class="f-grid">
  <div class="f-card">
    <h4>App</h4>
    <p>Google Play Store / Apple App Store / 원스토어에 앱이 등록되어 있어야 합니다.</p>
  </div>
  <div class="f-card">
    <h4>WEB</h4>
    <p>자체 콘텐츠를 보유한 웹 사이트여야 합니다. ads.txt 파일 업로드가 가능하며, Google 크롤러 접근이 허용되어야 합니다.</p>
  </div>
</div>

---

## 2. Google MCM 연동 절차

### Step 1. Google Ad Manager 계정 정보 전달

Google Ad Manager 계정으로 사용할 / 사용하는 <span class="hl">지메일 계정</span>을 전달해주세요.

AdSense, AdMob을 사용하여 수익화를 진행하고 있다면, 해당 <span class="hl">지메일 계정</span>을 전달해주세요. Ad Manager 계정이 없으신 경우는 초대장 링크를 통해 가입 진행 부탁드립니다.

### Step 2. MCM 초대장 확인 및 수락

전달 주신 메일주소로 MCM 초대장을 발송 드립니다. 메일 확인 후 수락해주세요.

<div class="callout warn">
  <strong>⚠️ Ad Manager 신규 가입 시 인증 절차 안내</strong>
  <p>Ad Manager 신규 가입의 경우 계정 본인확인을 위한 인증 절차가 필요합니다. 가입 후 인증 절차 진행해주세요.</p>
  <ul>
    <li><a href="https://support.google.com/admanager/answer/13985965?hl=ko" target="_blank">본인 인증</a> 진행 (1&#126;2일 소요) 후 기입하신 주소로 <a href="https://support.google.com/admanager/answer/1251398?hl=ko" target="_blank">우편 인증</a> PIN이 자동 발송됩니다. (3&#126;4주 소요)</li>
    <li>주소 입력 시, 1개의 층으로만 기입해주세요. <br><em>예) 2, 3, 4층 ❌ → 3층 ✅</em></li>
    <li>MCM 퍼블리셔는 수익금 발생 여부와 관계없이 계정 생성 시에 우편 PIN 인증 절차를 진행합니다.</li>
  </ul>
</div>

### Step 3. sellers.json 업데이트를 위한 정보 전달

나스미디어 도메인의 sellers.json에 퍼블리셔의 정보 업데이트가 필요합니다.

Ad Manager 로그인 → 지급 → 결제정보 → 설정 관리 메뉴에서 <span class="hl">조직 이름 또는 이름</span>을 전달 부탁드립니다.

### Step 4. app-ads.txt / ads.txt 업데이트

<div class="callout warn">
  <strong>⚠️ 필수 진행 사항</strong>
  <p>App / Web site 승인을 위해 필요한 부분이기에 반드시 진행되어야 합니다.</p>
</div>

- **WEB** : 사이트 도메인에 Ads.txt 등록
- **APP** : 스토어 내 등록된 웹사이트에 App-ads.txt 등록

파일은 루트 도메인에서 업로드 및 [robots.txt에서 크롤링을 허용](https://support.google.com/admanager/answer/6023741?hl=ko)이 필요합니다.

- **Google app-ads.txt / ads.txt** : 운영팀에 요청
- **nap mx app-ads.txt / ads.txt** : [파트너 사이트 → 지원센터 → 가이드 → ads.txt 내용 적용](https://publisher.admixer.co.kr/board/guide/ads)

### Step 5. App / Web Site 승인

구글 수익화를 위해 사이트/앱 승인이 필요하며, 운영팀에 수익화 진행할 **사이트 URL**과 **앱 패키지**를 전달 부탁드립니다. (승인 완료까지 3&#126;4일 소요)

App은 승인 후 운영팀에서 SDK 연동에 필요한 **Google App ID**를 전달드릴 예정입니다.

---

<div class="callout info">
  <strong>📧 문의</strong>
  <p>Google MCM 연동 관련 문의는 <a href="mailto:nap_mx@nasmedia.co.kr">nap_mx@nasmedia.co.kr</a>로 연락주세요.</p>
</div>
