# Vercel + Supabase Passkey Portfolio

## 1. Vercel
`index.html`을 정적 사이트의 루트로 배포합니다.
HTML 안의 `YOUR_SUPABASE_URL`과 `YOUR_SUPABASE_PUBLISHABLE_KEY`를 실제 값으로 교체합니다.
Service role/secret key는 절대 HTML에 넣지 않습니다.

## 2. Supabase
Dashboard에서 Authentication → Passkeys를 켭니다.

WebAuthn 설정:
- Relying Party Display Name: My Portfolio
- Relying Party ID: 실제 Vercel 도메인의 bare domain
- Relying Party Origins: 실제 `https://...` 주소

Supabase의 Passkey API는 현재 experimental이므로 프로젝트에서 passkey를 명시적으로 활성화해야 합니다.

SQL Editor에서 `schema.sql` 실행 → 두 테스트 계정의 UUID를 넣어 가상 자료를 추가합니다.

Edge Function:
`supabase/functions/private-notes/index.ts`
를 `private-notes` 함수로 배포합니다.
기본 JWT 검증을 유지합니다.

## 3. 초기 패스키 등록
Supabase Passkey 등록은 기존의 confirmed non-anonymous Auth 사용자가 필요합니다.
따라서 제출용 공개 URL에는 회원가입/비밀번호 입력 화면을 넣지 않고,
테스트 계정은 Supabase Dashboard에서 가상 이메일 계정으로 만든 뒤 일회성 초기 세션을 확보하여
`registerPasskey()`로 패스키를 등록합니다.
그 이후 실제 소개 페이지에서는 비밀번호 없이 `signInWithPasskey()`만 사용합니다.

두 번째 계정도 같은 방식으로 별도 패스키를 등록하여 계정 간 접근 통제를 시험합니다.

## 4. 확인
1. 시크릿 창에서 공개 소개 페이지가 보이고, 비공개 자료는 보이지 않아야 합니다.
2. 인증 전 `private-notes` GET 요청은 401이어야 합니다.
3. 패스키 로그인 후 200과 본인 소유 자료만 보여야 합니다.
4. 두 번째 계정의 UUID를 URL/body에 넣어도 서버는 그 값을 사용하지 않고 JWT의 `auth.uid()`만 사용합니다.
5. 같은 패스키를 삭제한 뒤에는 해당 패스키로 로그인할 수 없고, 남은 패스키로는 로그인할 수 있어야 합니다.

## 5. 제출문에 기록할 것
- 등록 시 공개키가 저장되고 개인키가 브라우저/서버로 전송되지 않았다는 WebAuthn 기록
- 로그인마다 새 challenge가 생성되고 재사용 challenge가 거절되는 기록
- 성공/실패 로그인 요청
- 인증 전 401과 인증 후 200
- 두 계정 간 접근 시도와 401/403 또는 본인 자료만 반환되는 기록
- 두 패스키 등록 → 하나 삭제 → 남은 패스키 로그인 성공 / 삭제한 패스키 실패
- 세션/토큰 값은 제출문에서 마스킹
