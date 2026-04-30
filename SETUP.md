# 댓글 기능 활성화 가이드 (Firebase Firestore)

이 사이트의 댓글은 Google Firebase Firestore에 저장됩니다. 무료 한도 안에서 충분히 사용할 수 있어요 (하루 50,000회 읽기, 20,000회 쓰기).

## 1. Firebase 프로젝트 만들기 (2분)

1. https://console.firebase.google.com/ 접속 → Google 계정 로그인
2. **프로젝트 추가** 클릭
3. 프로젝트 이름: `art-gallery` (원하는 이름 입력)
4. Google Analytics는 **사용 안 함** 선택 (필요 없음)
5. **프로젝트 만들기**

## 2. 웹 앱 등록 (30초)

1. 프로젝트 대시보드에서 **`</>`** (웹 아이콘) 클릭
2. 앱 닉네임: `art-gallery-web`
3. **Firebase 호스팅 설정** 체크박스는 **해제** (이미 GitHub Pages 사용 중)
4. **앱 등록** 클릭
5. 화면에 나오는 `firebaseConfig` 객체를 **그대로 복사**해서 저에게 알려주세요. 형태:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "art-gallery-xxxxx.firebaseapp.com",
  projectId: "art-gallery-xxxxx",
  storageBucket: "art-gallery-xxxxx.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc..."
};
```

> 이 값들은 공개되어도 안전합니다 (Firebase 웹 SDK는 클라이언트에서 동작하도록 설계됨). 실제 보안은 다음 단계의 보안 규칙이 담당합니다.

## 3. Firestore Database 만들기 (1분)

1. 좌측 메뉴 → **빌드** → **Firestore Database** 클릭
2. **데이터베이스 만들기** 클릭
3. 위치: **asia-northeast3 (서울)** 선택
4. **프로덕션 모드에서 시작** 선택 → **사용 설정**

## 4. 보안 규칙 붙여넣기 (1분)

1. Firestore 페이지 상단 탭에서 **규칙** 클릭
2. 기존 내용을 **모두 지우고** 아래를 붙여넣기:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /comments/{commentId} {
      allow read: if true;
      allow create: if
        request.resource.data.keys().hasOnly(['workId', 'name', 'content', 'createdAt']) &&
        request.resource.data.workId is string &&
        request.resource.data.workId.size() > 0 &&
        request.resource.data.workId.size() <= 200 &&
        request.resource.data.name is string &&
        request.resource.data.name.size() > 0 &&
        request.resource.data.name.size() <= 30 &&
        request.resource.data.content is string &&
        request.resource.data.content.size() > 0 &&
        request.resource.data.content.size() <= 500 &&
        request.resource.data.createdAt == request.time;
      allow update, delete: if false;
    }
  }
}
```

3. **게시** 클릭

이 규칙은:
- **읽기**: 누구나 모든 댓글을 볼 수 있음
- **작성**: 정해진 형식 (작품ID, 이름 1~30자, 내용 1~500자) 만 허용
- **수정/삭제**: 클라이언트에서는 불가 (스팸 삭제는 Firebase 콘솔에서 직접)

## 5. 설정 완료 후

2단계에서 복사한 `firebaseConfig` 객체를 알려주시면 제가 `index.html`에 붙여넣고 다시 배포해드립니다.

## 운영 팁

- **악성 댓글 삭제**: Firebase Console → Firestore Database → comments 컬렉션에서 직접 삭제
- **사용량 모니터링**: 좌측 메뉴 → 사용량 및 결제
- **댓글이 너무 많아지면**: 무료 한도 초과 시 자동으로 차단되며 결제 청구되지 않음
