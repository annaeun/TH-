# 태국어 리스닝 학습 노트 — 배포 가이드

## 1. Firebase 프로젝트 준비
1. https://console.firebase.google.com 에서 새 프로젝트 생성 (또는 기존 프로젝트 사용).
2. **Authentication → Sign-in method**에서 "Google" 로그인 제공업체를 사용 설정.
3. **프로젝트 설정 → 일반 → 내 앱**에서 웹 앱을 추가하고, 발급된 설정값을
   `index.html` 상단의 `FIREBASE_CONFIG` 객체에 그대로 붙여넣기:
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "...",
     authDomain: "...",
     projectId: "...",
   };
   ```
4. 구글 계정이 있으면 누구나 로그인해서 이 도구를 쓸 수 있습니다 (관리자 승인
   절차 없음). 로그인하지 않은 방문자에게는 로그인 버튼만 있는 빈 화면만 보이고,
   실제 도구 레이아웃/기능은 로그인 전까지 노출되지 않습니다.

## 2. Gemini API 키 발급 및 보호 (중요)
1. https://aistudio.google.com/app/apikey 에서 API 키를 발급받으세요.
2. **`index.html`에는 키를 붙여넣지 않습니다.** 로그인한 뒤, 앱 화면 상단의
   "Gemini API 키" 입력창에 직접 붙여넣고 "저장" 버튼을 누르면 됩니다. 이 값은
   `localStorage`를 통해 **각자의 브라우저에만** 저장되며, 배포되는 정적 파일(HTML/JS)
   안에는 키가 전혀 포함되지 않습니다. 즉 누군가 페이지 소스를 보더라도 키를 알 수
   없고, 로그인한 다른 사람과도 공유되지 않습니다 — 각자 자기 키를 넣고 자기 브라우저에서만
   씁니다. (새 브라우저·기기·시크릿창에서는 다시 입력해야 하고, 브라우저 저장공간을
   지우면 사라집니다.)
3. Google Cloud Console → API 및 서비스 → 사용자 인증 정보에서 해당 키를 선택해
   "애플리케이션 제한사항 → HTTP 리퍼러(웹사이트)"로 설정하고, 실제 쓸 도메인(예:
   `https://YOUR_PROJECT_ID.web.app/*`, `http://localhost:5173/*`)만 허용해두면
   더 안전합니다.

## 3. Firestore 활성화 (분석 히스토리용)

이 도구는 로그인한 사용자의 분석 히스토리를 저장하기 위해 Firestore를 사용합니다
(사용자 승인 절차는 없습니다 — 로그인한 누구나 바로 이용할 수 있습니다).

1. **Firestore 활성화** (아직 안 했다면): Firebase 콘솔 → Firestore Database →
   데이터베이스 만들기 (프로덕션 모드로 시작해도 무방 — 아래 규칙으로 덮어씁니다).
2. **보안 규칙 적용**: Firestore Database → 규칙 탭에서, 이 프로젝트의
   `firestore.rules` 파일 내용을 그대로 붙여넣고 게시(Publish)하세요.

## 4. 분석 히스토리 자동 삭제 (Firestore TTL)

로그인한 사용자가 분석한 결과는 `history` 컬렉션에 저장되고, 6개월이 지나면
자동으로 삭제되도록 설계되어 있습니다. 자동 삭제가 실제로 동작하려면 Firestore에
TTL(Time-to-live) 정책을 한 번 등록해야 합니다.

1. Firebase 콘솔 → Firestore Database → **TTL** 탭 → "정책 추가"
2. 컬렉션 그룹 ID: `history`
3. 타임스탬프 필드: `expireAt`
4. 저장

등록해두면 `expireAt`이 지난 문서가 자동으로 삭제됩니다 (정확히 그 순간이 아니라
보통 하루~이틀 이내에 처리되며, 이건 Firestore 자체의 정상적인 동작 방식입니다).
등록하지 않아도 앱은 정상 동작하지만, 오래된 기록이 계속 쌓이게 됩니다.

`firestore.rules`를 갱신했다면 (`history` 컬렉션 규칙 추가), 콘솔의 규칙 탭에 다시
붙여넣거나 `firebase deploy --only firestore:rules`로 반영해주세요.

## 5. 로컬 미리보기
```bash
npx serve .
```
(Google 로그인 팝업은 `localhost`에서도 정상 동작합니다. Firebase 콘솔의
Authentication → Settings → 승인된 도메인에 `localhost`가 기본 포함되어 있는지 확인하세요.)

## 6. 배포
```bash
npm install -g firebase-tools   # 최초 1회
firebase login
firebase init hosting           # 기존 프로젝트 선택, public 디렉터리는 "." 그대로 사용
firebase deploy --only hosting
firebase deploy --only firestore:rules   # firestore.rules 반영 (콘솔에서 이미 붙여넣었다면 생략 가능)
```

## 7. 사용 방법

실제 화면 조작법(언어 선택, 파일 업로드, 단어 추가, 복사하기 등)은
[README.md](./README.md)의 "사용법" 섹션을 참고하세요. 이 문서는 배포/운영 설정에
집중합니다.
