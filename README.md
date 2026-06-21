📘 Maxerve SMART 웹앱 상세 설명서
1. 📌 개요

Maxerve SMART는
현장에서 설비 점검을 수행하기 위한 모바일 기반 웹앱(PWA)이다.

주요 기능:

사용자 인증 (이름 + 전화번호)
QR/NFC 기반 설비 식별
동적 점검표 생성
사진 첨부 및 메모 기록
Google Apps Script 서버로 데이터 전송
2. 🏗️ 전체 구조
📁 구성 파일
파일명	설명
index.html	메인 앱 (UI + 로직 포함)
manifest.json	PWA 설정 파일
3. 🔐 인증 시스템
3.1 인증 흐름
사용자 이름 + 전화번호 입력
AUTH_URL로 요청 전송
구글 시트 기반 인증 확인
성공 시 → 센터명 반환
로컬스토리지 저장
localStorage.setItem('userCenter', centerName);
localStorage.setItem('userName', name);
localStorage.setItem('userPhone', phone);
3.2 인증 상태 유지

앱 시작 시 자동 로그인 처리:

window.onload = function() {
    if (savedCenter && savedName && savedPhone) {
        startInspection(...)
    }
}

👉 즉, 한 번 로그인하면 계속 유지됨

4. 📡 점검 흐름
전체 플로우
[로그인]
   ↓
[스캔 대기]
   ↓
[QR/NFC 인식]
   ↓
[설비 데이터 로드]
   ↓
[점검 입력]
   ↓
[사진 첨부]
   ↓
[데이터 전송]
   ↓
[완료 → 다시 스캔]
5. 📷 QR 스캔 시스템
사용 라이브러리
html5-qrcode
<script src="https://unpkg.com/html5-qrcode"></script>
동작 방식
html5QrCode.start(
  { facingMode: "environment" },
  { fps: 20, qrbox: 250 },
  (text) => {
    location.href = text;
  }
);

👉 QR 코드 안에 URL이 들어있어야 정상 동작

6. 📊 설비 데이터 로딩
요청 구조
GET webAppUrl?fid=설비ID
             &centerName=센터명
             &worker=이름
             &phone=번호
서버 응답 구조 (JSON)
{
  "facilityName": "펌프 A",
  "category": "기계",
  "configString": "온도|number;상태|select|정상,이상"
}
7. 🧩 동적 점검표 생성
핵심 구조
configString.split(';')

각 항목 구조:

이름 | 타입 | 옵션
타입 종류
타입	설명
text	텍스트 입력
number	숫자 입력
select	선택
info	설명
line	구분선
예시
온도|number
상태|select|정상,이상
점검내용|text
안내|info|이 설비는 중요함
구분|line
8. 📸 사진 처리 시스템
기능
최대 3장 제한
자동 압축
Base64 변환
압축 로직
canvas.toDataURL('image/jpeg', 0.6);

👉 용량 줄여서 서버 부담 최소화

저장 방식
photoData.push(base64데이터);
9. 📤 데이터 전송
POST 요청 구조
{
  "centerName": "울산센터",
  "worker": "홍길동",
  "fId": "설비ID",
  "memo": "내용",
  "photoData": [...],
  "values": [...]
}
특징
최대 30개 필드 고정 배열
빈 값은 "" 처리
let finalValues = new Array(30).fill("");
10. 🔄 완료 후 동작
완료 시 처리
SUCCESS 확인
완료 UI 표시
3초 후 자동 이동
URL 파라미터 제거
window.history.replaceState(...)

👉 같은 설비 재입력 방지

11. 💾 로컬스토리지 구조
키	내용
userCenter	센터명
userName	사용자 이름
userPhone	전화번호
completed_form_ID	점검 완료 기록
12. 🎨 UI 구조
주요 화면
화면	ID
로그인	step-auth
스캔 대기	step-scan
점검폼	inspection-form
로딩	loading-layer
13. 📱 PWA 설정

manifest.json 구성:

{
  "display": "standalone",
  "start_url": "./index.html"
}

👉 앱처럼 실행 가능

14. ⚙️ 주요 특징 요약
✅ 강점
서버 의존 최소화
모바일 최적화
QR 기반 자동화
동적 폼 생성
사진 압축 기능
⚠️ 주의사항
QR 코드 URL 정확해야 함
구글 스크립트 응답 형식 중요
로컬스토리지 초기화 시 재로그인 필요
사진 3장 제한
15. 🔧 확장 가능 기능
NFC 태그 지원 강화
오프라인 저장 후 동기화
점검 이력 조회 기능
관리자 대시보드
Firebase 연동
🔚 결론

이 시스템은 단순 웹페이지가 아니라:

👉 현장 점검 자동화 플랫폼

구성 핵심은 딱 3개:

QR → 동적폼 → 서버전송
