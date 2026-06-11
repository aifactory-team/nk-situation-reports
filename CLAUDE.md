# 원자력통제기술원 북한 핵 상황보고

## 프로젝트 목적
네이버뉴스 정치>북한 섹션을 매일 수집하여 KINAC 4대 카테고리(지역/핵물질생산/핵주기/핵주기시설)로 분류한 일일 상황보고서를 생성한다.

## 하네스: NK 상황보고

**목표:** 단일 소스(네이버 정치>북한)·한국어 4키워드·KINAC 4대 카테고리로 좁힌 일일 북한 핵 상황보고 자동화. (`north-korean-nuclear-activities`의 광범위 파이프라인을 좁힌 버전)

**트리거:** 북한 핵 상황보고/KINAC 보고서 관련 작업(생성·재실행·업데이트) 요청 시 `nk-situation-report` 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**파이프라인:** 수집(nk-collector) → 분류(nk-classifier) → 분석(nk-analyst) → 보고서(nk-reporter). 순차 서브 에이전트 파이프라인, 파일 핸드오프.

**설정:** `config/search-config.json` (수집 대상·키워드·4대 카테고리). 범위 변경 시 이 파일만 편집.

**실행 환경:** 자동 실행은 GitHub Actions(`.github/workflows/daily-nk-situation-report.yml`)에서 매일 KST 08:00에 수행. 네이버 직접 fetch는 차단되므로 [cheliped-browser](https://github.com/tykimos/cheliped-browser)를 GHA 러너에 설치해 `$CHELIPED_CLI`로 정치>북한 섹션을 크롤링한다. 필요 시크릿: `CLAUDE_CODE_OAUTH_TOKEN`.

## 디렉토리
```
config/search-config.json     # 수집 대상 + KINAC 분류 기준
sources/YYYY-MM-DD/           # 파이프라인 중간 산출물(추적용)
  search-results.json          # Phase1 수집
  index.json + items/          # Phase2 분류·태깅
  analysis.md + report-basis.md# Phase3 분석
reports/YYYY/MM/YYYY-MM-DD.md # Phase4 최종 보고서
```

## 규칙
- 출처 URL 없는 정보는 보고서에 포함하지 않는다
- 보고서는 한국어, KINAC 4대 카테고리 구조를 항상 유지(해당 없으면 "해당 없음")
- 포함 항목이 0건이어도 보고서 파일은 생성한다
- 파이프라인 중간 산출물은 항상 생성하고 함께 커밋한다(`git add sources/ reports/`)

## 커밋 컨벤션
- 보고서: `report: KINAC NK situation report (YYYY-MM-DD)`
- 구조/설정: `chore: 설명`

## 변경 이력
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-06-11 | 초기 구성 (north-korean-nuclear-activities에서 이식·좁힘) | 전체 | KINAC 요구: 네이버 정치>북한 단일소스 + 4대 카테고리 분류 |
| 2026-06-12 | GHA 워크플로 추가 (cheliped-browser 설치·크롤링) | .github/workflows | 자동 일일 실행 + 네이버 크롤링 차단 우회 |
| 2026-06-12 | 기술 문서 작성 + actions 버전 상향(@v5/Node22) | docs/ARCHITECTURE.md, .github/workflows | 개념도·시퀀스 등 기술 문서화, Node20 deprecation 대응 |
| 2026-06-12 | 위키 자동 발행 단계 추가 | .github/workflows | 매일 보고서를 GitHub Wiki(Home/_Sidebar/Monthly)로 발행. 위키 첫 페이지는 웹 UI 1회 생성 필요 |
