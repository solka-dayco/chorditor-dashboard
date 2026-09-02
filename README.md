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

정적 파일이라 브라우저로 그냥 열면 된다. 페이지가 1분마다 스스로 데이터를 다시 불러온다.

`G` 키를 누르면 그리드 오버레이가 켜진다(컬럼 정렬 확인용).

## 주의

Supabase anon key가 파일에 그대로 들어있다. 이 키는 앱 번들에도 있어 이미 공개된 값이지만,
`dau_wau_mau_history` 테이블에 anon SELECT 정책이 열려 있으므로 저장소는 private으로 둔다.
