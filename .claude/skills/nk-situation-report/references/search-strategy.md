# 수집 전략 (좁은 범위 — KINAC)

## 단일 소스
- **네이버뉴스 정치 > 북한 섹션** 한 곳만 수집한다.
- 섹션 URL: `https://news.naver.com/breakingnews/section/100/268` (정치=100, 북한=268)
- `config/search-config.json`의 `source`를 기준으로 한다.
- **금지:** 타 섹션, 영어/일본어/중국어, 타 검색엔진(Google/Bing 등), 해외 사이트(38 North/NK News 등). 이것이 소스 레포 대비 좁힌 핵심이다.

## 1차 필터 키워드 (한국어)
`북한 핵`, `북핵`, `핵실험`, `핵물질`
- 제목·요약·본문 중 하나라도 매칭되면 수집 대상.
- 하나도 매칭되지 않으면 버린다.

## 수집 방법

### 섹션 크롤링
섹션 페이지에서 당일(또는 대상일) 기사 목록을 추출한 뒤, 각 기사의 제목·URL·스니펫·매체명·게재시각을 수집한다.

**WebFetch 사용:**
```
WebFetch(url="https://news.naver.com/breakingnews/section/100/268",
         prompt="이 페이지의 기사 목록에서 제목, 기사 URL, 매체명, 게재 시각을 모두 추출하라")
```

**브라우저 CLI(JS 렌더링·더보기 페이지네이션 필요 시):**
```bash
node $CHELIPED_CLI '[{"cmd":"goto","args":["https://news.naver.com/breakingnews/section/100/268"]},{"cmd":"wait","args":["2000"]},{"cmd":"extract","args":["all"]},{"cmd":"close"}]'
```
- 반드시 `close`로 세션 종료.
- 개별 기사 본문이 필요하면 기사 URL로 한 번 더 `goto`/`extract`.

## 출력 스키마: search-results.json
```json
{
  "date": "YYYY-MM-DD",
  "collected_at": "ISO8601",
  "source": "naver_politics_nk",
  "total_results": 12,
  "filtered_out": 30,
  "results": [
    {
      "title": "기사 제목",
      "url": "https://n.news.naver.com/...",
      "snippet": "기사 요약/발췌",
      "source_name": "매체명",
      "published_at": "YYYY-MM-DD HH:MM",
      "matched_keywords": ["북핵", "핵실험"]
    }
  ]
}
```
- `matched_keywords`: 어떤 필터 키워드에 걸렸는지 기록(추적용).
- 이 파일은 write-only(이후 Phase에서 재읽기 않음, 디버깅/추적 전용).
