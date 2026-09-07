# GitHub Pages + Supabase 게시판

## 1. Supabase
프로젝트를 만든 뒤 Dashboard → SQL Editor에서 `schema.sql` 전체를 실행합니다.

## 2. 관리자 계정 생성 및 권한 설정
로그인 화면에서 이메일 전체 대신 **아이디(예: `wondora`)**만 입력하여 로그인할 수 있습니다.
(내부적으로 `아이디@admin.local` 형태로 Supabase Auth에 인증됩니다. 필요 시 `index.html`의 `ADMIN_EMAIL_DOMAIN`에서 변경 가능)

1. Supabase Dashboard → **Authentication** → **Users** → **Add user** (Create user)
   - **Email**: `wondora@admin.local` (아이디 뒤에 `@admin.local`을 붙여 생성)
   - **Password**: 원하는 비밀번호 입력
   - **Auto Confirm User?**: 체크 활성화

2. Supabase Dashboard → **SQL Editor**에서 관리자 권한(`role: admin`) 부여:
   ```sql
   UPDATE auth.users
   SET raw_app_meta_data = coalesce(raw_app_meta_data,'{}'::jsonb) || '{"role":"admin"}'::jsonb
   WHERE email = 'wondora@admin.local';
   ```

3. 게시판 화면의 [관리자] 로그인:
   - **아이디**: `wondora` (또는 `wondora@admin.local` 입력 가능)
   - **비밀번호**: 설정한 비밀번호 입력

## 3. API 정보
Supabase Dashboard → Project Settings → API에서 Project URL과 anon/public key를 확인합니다.
`index.html`의
SUPABASE_URL
SUPABASE_ANON_KEY
두 값을 바꿉니다.

service_role key는 절대 넣지 마세요.

## 4. GitHub Pages
GitHub repository에 index.html을 올립니다.
Settings → Pages → Deploy from branch → main / root.

## 현재 기능
- **일반 방문자**: 게시글 목록 조회, 검색, 카테고리 분류, 게시글 상세 조회 (댓글 기능 없음)
- **관리자**: 
  - 아이디 기반 관리자 로그인/로그아웃
  - 게시글 작성 (`insert`)
  - 게시글 수정 (`update`)
  - 게시글 삭제 (`delete`)
- **에디터**: 서식 툴바(굵게, 기울임, 밑줄, 목록, 링크), 캡처 이미지 붙여넣기(Ctrl+V), 자료 바로가기 링크

## 이미지 관련
현재 캡처 이미지는 별도 파일 저장소가 아니라 본문에 data URL로 저장합니다.
작은 캡처에는 편하지만 대형 이미지를 많이 올리면 DB가 커질 수 있습니다.
운영을 시작한 뒤 필요하면 Supabase Storage 방식으로 교체하는 것을 권장합니다.

--- 

방법 1. Supabase SQL Editor에서 비밀번호 즉시 재설정 (가장 추천)
Supabase 대시보드 → SQL Editor에서 아래 쿼리의 '원하는비밀번호' 부분만 바꾸어 실행(Run)하시면, 비밀번호가 즉시 변경되며 이메일 인증과 관리자 권한까지 한 번에 완벽히 세팅됩니다.

sql


-- 비밀번호를 새로 지정하고 이메일 인증 및 관리자 권한을 부여합니다.
UPDATE auth.users
SET encrypted_password = crypt('원하는비밀번호', gen_salt('bf')),
    email_confirmed_at = coalesce(email_confirmed_at, now()),
    raw_app_meta_data = coalesce(raw_app_meta_data, '{}'::jsonb) || '{"role":"admin"}'::jsonb
WHERE email = 'coswons@admin.local';
TIP

위 쿼리를 실행한 뒤, 실행 결과에 Success. No rows returned 또는 UPDATE 1이 뜨면 바로 게시판으로 가셔서 coswons와 새로 설정한 비밀번호로 로그인해 보세요!

방법 2. 계정 상태 확인 쿼리
만약 위 쿼리를 실행했는데 UPDATE 0(영향받은 행이 0개)이 뜬다면, coswons@admin.local이라는 계정 자체가 아직 생성되지 않았거나 이메일 스펠링이 다르게 등록된 상태입니다.

아래 쿼리로 등록된 유저 목록을 확인해 보실 수 있습니다:

sql


SELECT id, email, email_confirmed_at, raw_app_meta_data 
FROM auth.users;
방법 3. 대시보드에서 계정 지우고 다시 만들기
Supabase Authentication → Users 목록에서 coswons@admin.local 우측의 ⋯ 버튼을 눌러 Delete user로 삭제합니다.
다시 Add user → Create user를 누르고:
Email: coswons@admin.local
Password: 단순하고 확실한 비밀번호 (예: 12345678 또는 본인 비밀번호)
Auto Confirm User?: 체크 (ON)
생성 후 SQL Editor에서 관리자 권한 쿼리 실행:
sql


UPDATE auth.users
SET raw_app_meta_data = coalesce(raw_app_meta_data, '{}'::jsonb) || '{"role":"admin"}'::jsonb
WHERE email = 'coswons@admin.local';
