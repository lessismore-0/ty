# good day 건강앱 작업 규칙

개인용 건강 대시보드입니다. 이 저장소에서 작업하는 Claude는 아래 규칙을 따릅니다.
사용자는 코드를 직접 고치지 않고 결과만 확인합니다. 설명은 한국어로, 초보자도 따라 할 수 있게 순서대로 합니다.

## 1. 구성
- 웹: `good.html` 한 파일(HTML/CSS/JS). GitHub Pages로 배포: https://lessismore-0.github.io/ty/good.html
- 아이폰 동기화 앱(Xcode, HealthKit 읽기 전용 → Supabase 업로드). 이 저장소에는 없음
- Supabase: `public.health_samples`, `public.health_sync_runs` (RLS 적용)

## 2. 작업 방식 (중요)
- **수정 전에 항상 GitHub의 최신 main을 받아서 그 위에 고친다.** 사용자가 폰이나 GitHub 웹에서 good.html을 직접 바꿀 수 있으므로, 예전 사본으로 덮어쓰면 안 된다.
- 사용자가 웹 수정을 main에 바로 반영하도록 허락함(2026-11-15). 확인 → 수정 → 브라우저 테스트 → main에 커밋·push 후, 바뀐 점과 확인할 곳을 짧게 알린다.
- 이전 버전은 git 이력이 백업이다. 되돌려 달라고 하면 해당 커밋의 good.html로 되돌린다. 브랜치 `good-html-sync-mood-coffee-delete`는 백업으로 남겨둔다(삭제 금지).
- 테스트: 사전 설치된 Playwright로 good.html을 열고, 시계를 고정(`page.clock.setFixedTime`)하고 타임존 `Asia/Seoul`로 확인한다. 휴대폰 너비(390px) 화면도 본다. 페이지 오류(pageerror)가 없어야 한다.
- **개인 데이터는 절대 커밋하지 않는다.** 커피·기분 백업 JSON, 건강 기록, 계정 정보, 토큰 등. 이 저장소는 공개다.
- 아이폰 앱 코드를 줄 때는 항상 **전체 파일**로 준다(사용자는 통째로 복붙만 함). 작업 순서: SQL → 아이폰 앱 → 웹.

## 3. 바꾸면 안 되는 것
- 배포 주소와 파일 이름 `good.html`. 파일은 통째로 교체하는 방식
- 웹 저장 키: `harugyeol-coffee-v1`(커피), `harugyeol-moods-v1`(기분), `good-day-water-v1`(물), `good-day-auth-config-v1`(인증 설정)
- 백업 형식: `format: "good-day-backup"`, `version: 2`, 커피·기분·즐겨찾기만 포함. 불러오기 시 같은 id는 건너뜀, 최대 50,000건, 20MB 이하
- 앱 딥링크: `gooddayhealth://` (로그인 콜백 `gooddayhealth://auth/callback`). Xcode URL Types에 등록돼 있음
- 앱 UserDefaults 키: `goodday.publicProject.v1`, `goodday.uploadConsent.v1`
- Supabase는 프로젝트 URL + `sb_publishable_` 공개 키만 사용 (secret·service_role 키 금지)
- 기존 테이블 삭제 금지

## 4. 확정된 규칙
- 카페인: 성인 1일 한도 400 mg. 400 초과만 "초과"(빨강), 정확히 400은 "한도 도달". 건강앱(서버) 카페인 기록이 있으면 그 값, 없으면 직접 기록. 두 값은 더하지 않음
- 수분: 목표 1,500 mL, 건강앱 기록만 합산 (과거 수동 기록은 보존하되 합계 제외)
- 탄단지: 건강앱 g 합계, kcal은 4·4·9 환산
- 기분: 5단계, 메모 200자, 최근 7일 그래프
- 수면 점수: 시간 70점 + 연속성 30점 (자체 지표, 의료용 아님). 겹친 구간은 한 번만, 저녁 6시 이후 시작한 잠은 다음 날로 묶음, 수면 범주 값 6은 수면으로 세지 않음
- 체성분: 체중은 측정값 그대로(소수점 최대 2자리), 체지방률은 소수점 1자리. 그래프는 하루 마지막 측정값 기준. 체중 그래프는 최근 7일(요일별 한 칸), 체지방률 그래프는 최근 30일. 주간 변화 막대그래프는 사용자 요청으로 삭제함(2026-11-15)
- 음료 DB: 사용자 음료정보 시트 2026-11-08 저장본, 169개 항목, 톨 사이즈 고정, `EXCLUDED_ROWS = Set([22,152,158,161])`. 142번째 "얼 그레이 티"는 이름이 추정값이라 확인 필요
  시트: https://docs.google.com/spreadsheets/d/1tX59YaFx8BHiXkOkoKdA7W7fok30VFH13VaRBInu3kE/edit?usp=sharing
- 차트: 단위가 다른 값은 한 그래프에 축 두 개로 넣지 않고 따로 그린다. 색은 CSS 변수(`--caf`, `--water`, `--weight`, `--bodyfat` 등)를 쓴다

## 5. 서버 (Supabase)
- kind: water(mL), weight(kg), body_fat(%), sleep, menstrual, caffeine(mg), carbs(g), protein(g), fat(g)
- 제약 `gd_health_kind_check`, `gd_health_value_check` 적용 완료(2026-11-13). 고유 키 `(user_id, kind, sample_id)`
- 원본 건강앱에서 삭제한 기록의 서버 반영은 미구현

## 6. 아이폰 앱 (2026-11-13 적용 완료)
- 조회 기간: 수면 9일, 체중·체지방·수분·카페인·탄단지 30일, 월경 90일
- 업로드 동의 버전 2, 문구 "아홉 종류". `ExportModels.swift`는 건드리지 않음
- 빌드 경고 8개(iOS 버전별 이름 변경 안내)는 동작에 문제없음
