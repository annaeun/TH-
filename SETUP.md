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
4. 로그인은 구글 계정이면 누구나 시도할 수 있지만, **실제로 도구를 쓰려면 관리자의
   승인**이 필요합니다 (승인 방법은 아래 "3. 사용자 승인 관리" 참고). 로그인하지 않은
   방문자에게는 로그인 버튼만 있는 빈 화면만 보이고, 실제 도구 레이아웃/기능은
   승인 전까지 전혀 노출되지 않습니다.

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

## 3. 사용자 승인 관리 (Firestore)

로그인은 누구나 시도할 수 있지만, Firestore의 `approvedUsers` 컬렉션에 등록된
이메일만 실제로 도구 화면을 볼 수 있습니다. 승인/차단 모두 **Firebase 콘솔에서
문서를 추가/삭제하는 것만으로** 처리되며, 별도 서버 코드나 이메일 발송 기능은 없습니다.

1. **Firestore 활성화** (아직 안 했다면): Firebase 콘솔 → Firestore Database →
   데이터베이스 만들기 (프로덕션 모드로 시작해도 무방 — 아래 규칙으로 덮어씁니다).
2. **보안 규칙 적용**: Firestore Database → 규칙 탭에서, 이 프로젝트의
   `firestore.rules` 파일 내용을 그대로 붙여넣고 게시(Publish)하세요.
3. **본인(관리자) 계정부터 승인 등록** — 이걸 안 하면 본인도 접근이 막힙니다:
   - Firestore Database → 데이터 탭 → "컬렉션 시작" → 컬렉션 ID: `approvedUsers`
   - 문서 ID: 본인 구글 이메일 주소를 정확히 입력 (예: `nimankarai2@gmail.com`)
   - 필드 추가: 필드 이름 `approved`, 유형 `boolean`, 값 `true`
   - 저장
4. **새 사용자 승인하기**: 누군가 로그인을 시도하면 `requests` 컬렉션에 그 사람의
   이메일로 문서가 자동 생성됩니다 (요청 시각 포함). 그 이메일을 확인한 뒤,
   3번과 같은 방식으로 `approvedUsers` 컬렉션에 같은 이메일 문서를 만들고
   `approved: true`를 넣으면 그 사람도 즉시 이용할 수 있게 됩니다.
5. **승인 취소**: `approvedUsers`에서 해당 이메일 문서를 삭제하거나 `approved`를
   `false`로 바꾸면 됩니다.

⚠️ 이 방식은 관리자가 가끔 Firebase 콘솔의 `requests` 컬렉션을 직접 확인해야
합니다 (새 요청이 와도 이메일 알림 등은 오지 않습니다). 나중에 앱 안에서 바로
승인하는 패널이나, 새 요청 시 이메일 알림을 받고 싶으시면 언제든 추가해드릴 수
있습니다 (다만 이메일 알림은 Firebase 유료 요금제 전환이 필요합니다).

## 4. 로컬 미리보기
```bash
npx serve .
```
(Google 로그인 팝업은 `localhost`에서도 정상 동작합니다. Firebase 콘솔의
Authentication → Settings → 승인된 도메인에 `localhost`가 기본 포함되어 있는지 확인하세요.)

## 5. 배포
```bash
npm install -g firebase-tools   # 최초 1회
firebase login
firebase init hosting           # 기존 프로젝트 선택, public 디렉터리는 "." 그대로 사용
firebase deploy --only hosting
firebase deploy --only firestore:rules   # firestore.rules 반영 (콘솔에서 이미 붙여넣었다면 생략 가능)
```

## 6. 사용 방법

실제 화면 조작법(언어 선택, 파일 업로드, 단어 추가, 복사하기 등)은
[README.md](./README.md)의 "사용법" 섹션을 참고하세요. 이 문서는 배포/운영 설정에
집중합니다.
