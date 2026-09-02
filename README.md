# Chorditor Dashboard

Chorditor 앱의 지표를 보는 내부 전용 정적 대시보드. 앱 저장소(Chorditor)와 분리해서 관리한다.

## 구성

| 파일 | 내용 |
|---|---|
| `dashboard.html` | DAU / WAU / MAU / 신규가입 실시간 지표 + 추세 차트 |
| `analytics-projects.html` | 기능별 퍼널 등 분석 프로젝트 보드 |

## 데이터 흐름

Supabase의 집계 테이블 `dau_wau_mau_history`를 REST로 직접 읽는다.
집계 자체는 DB 안에서 pg_cron이 돌린다 — 페이지는 계산하지 않고 결과만 조회한다.

| 지표 | 갱신 주기 | 정의 |
|---|---|---|
| DAU | 5분 | 지금 기준 지난 24시간 distinct user_id |
| WAU | 1시간 | 지난 7일 |
| MAU | 1시간 | 지난 30일 |
| signups | 5분 | 당일(KST) `auth.users` 신규 생성 수 |

DAU/WAU/MAU는 sliding window라서, 자정 시점의 값이 그날 캘린더데이 값과 정확히 일치한다.
따라서 날짜가 바뀌면 그 행이 자동으로 확정값이 된다(별도 마감 처리 없음).

## 사용

GitHub Pages로 배포된 주소에서 연다. Google 계정으로 로그인해야 지표가 보이고,
로그인 후에는 페이지가 1분마다 스스로 데이터를 다시 불러온다.

`file://`로 직접 열면 OAuth 리다이렉트가 동작하지 않아 로그인할 수 없다.

`G` 키를 누르면 그리드 오버레이가 켜진다(컬럼 정렬 확인용).

## 접근 제어

저장소가 public이라 anon key가 그대로 노출된다. 이 키는 앱 번들에도 들어 있어 원래
공개 전제인 값이고, 여기서는 PostgREST가 요구하는 `apikey` 헤더 용도로만 쓴다.

실제 접근 권한은 로그인 세션이 결정한다.

- `dau_wau_mau_history`에서 anon SELECT 권한을 회수했다 — 로그인 없이는 아무것도 못 읽는다.
- 정책 `dau_wau_mau_history_select_admin`이 관리자 uid 하나만 통과시킨다.
  다른 구글 계정으로 로그인하면 세션은 생기지만 조회가 막힌다.
- 조회는 raw fetch에 세션 `access_token`을 실어 보낸다. supabase-js 클라이언트로 조회하면
  `auth.uid()`가 null로 잡혀 RLS에 막히는 문제가 있어 인증에만 쓴다.

관리자 uid를 바꾸거나 계정을 추가하려면 위 정책을 수정한다.

새 페이지를 추가해 DB를 읽게 만들 때는 같은 게이트를 반드시 같이 붙인다.
`analytics-projects.html`은 아직 네트워크 호출이 없어 게이트가 없다.
