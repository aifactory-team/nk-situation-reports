---
name: nk-situation-report
description: "원자력통제기술원(KINAC) 북한 핵 일일 상황보고서를 자동 생성하는 파이프라인 오케스트레이터. 네이버뉴스 정치>북한 섹션만 수집하고 KINAC 4대 카테고리(지역/핵물질생산/핵주기/핵주기시설)로 분류하는 좁은 범위 버전. 4단계(수집→분류→분석→보고서)로 sources/ 폴더에 추적 가능한 산출물을 남긴다. '북한 핵 상황보고', 'KINAC 보고서', '북핵 일일 보고서', '상황보고 생성', '다시 실행', '재실행', '업데이트', '오늘 북한 핵' 요청 시 사용."
---

# NK Situation Report — KINAC 파이프라인 오케스트레이터

네이버뉴스 정치>북한 섹션 단일 소스를 4단계 파이프라인으로 처리하여 KINAC 4대 카테고리 구조의 일일 상황보고서를 생성한다.
각 에이전트는 자신의 역할(`.claude/agents/`)과 참조 스킬(`references/`)을 읽고 작업한다.

> **이 하네스는 `north-korean-nuclear-activities`(광범위 검색)를 KINAC 요구에 맞춰 좁힌 버전이다.** 단일 소스(네이버 정치>북한) · 한국어 4키워드 · KINAC 4대 카테고리 분류가 핵심 차이다.

## 실행 모드
**순차 서브 에이전트 파이프라인** — 각 Phase가 앞 Phase의 파일 산출물에 의존하므로 순서대로 실행한다(파일 핸드오프). 각 에이전트는 `model: "opus"`로 호출한다.

## Phase 0: 컨텍스트 확인 & 준비
1. 대상 날짜 결정(미지정 시 오늘)
2. `sources/YYYY-MM-DD/`가 이미 존재하는지 확인:
   - 존재 + 부분 재실행 요청(예: "분류만 다시") → 해당 Phase만 재호출
   - 존재 + 새 수집 요청 → 기존을 `sources/YYYY-MM-DD_prev/`로 이동 후 새 실행
   - 미존재 → 초기 실행
3. `mkdir -p sources/YYYY-MM-DD/items`, `mkdir -p reports/YYYY/MM`
4. 이전 7일의 `sources/*/index.json`, `reports/YYYY/MM/*.md` 목록 조회

## Phase 1: 수집 (Collect)
**에이전트:** `.claude/agents/nk-collector.md`
**참조:** `references/search-strategy.md`
**산출물:** `sources/YYYY-MM-DD/search-results.json`
네이버 정치>북한 섹션 크롤링 + 4키워드 필터.

## Phase 2: 분류 (Classify)
**에이전트:** `.claude/agents/nk-classifier.md`
**참조:** `references/classification-rules.md`
**산출물:** `sources/YYYY-MM-DD/index.json` + `items/src-XXX.json`
KINAC 4대 카테고리 분류 + new/reported/update 중복제거 태깅.

## Phase 3: 분석 (Analyze)
**에이전트:** `.claude/agents/nk-analyst.md`
**참조:** `references/analysis-guide.md`
**산출물:** `sources/YYYY-MM-DD/analysis.md` + `report-basis.md`

## Phase 4: 보고서 (Report)
**에이전트:** `.claude/agents/nk-reporter.md`
**참조:** `references/report-format.md`
**산출물:** `reports/YYYY/MM/YYYY-MM-DD.md` (KINAC 4대 카테고리 구조)

## Phase 5: 커밋
```bash
git add sources/ reports/
git commit -m "report: KINAC NK situation report (YYYY-MM-DD)"
git push
```

## 에러 핸들링

| Phase | 에러 | 전략 |
|-------|------|------|
| 1 | 섹션 접근 실패 | 1회 재시도 → 빈 search-results.json, 계속 |
| 1 | 키워드 매칭 0건 | 빈 결과 → "특이사항 없음" 보고서로 이어짐 |
| 2 | 이전 index.json 없음 | 전체 `new`로 태깅 |
| 2 | 카테고리 모호 | 최근접 1개 + 사유 기록 |
| 3 | 이전 보고서 없음 | 신규 기사만으로 분석 |
| 4 | 포함 항목 0건 | "특이사항 없음" 보고서(4대 카테고리 섹션 유지) |
| 5 | push 실패 | 로컬 커밋 유지, 수동 push 안내 |

## 테스트 시나리오
- **정상 흐름:** 오늘 날짜로 전체 실행 → 섹션에서 핵 관련 기사 N건 수집 → 4대 카테고리 분류 + 태깅 → 분석 → 4대 카테고리 보고서 생성 → 커밋.
- **에러 흐름(특이사항 없음):** 섹션에 핵 키워드 기사 0건 → Phase 1 빈 결과 → Phase 2~3 스킵/간략 → Phase 4 "특이사항 없음" 보고서(4대 카테고리 섹션은 '해당 없음'으로 유지) 생성.
