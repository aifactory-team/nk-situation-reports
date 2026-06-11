# 분류 규칙 — KINAC 4대 카테고리 + 중복제거 태깅

각 기사에 **두 가지 라벨**을 독립적으로 부여한다: ① KINAC 카테고리(무엇에 관한 기사인가) ② 중복제거 태그(이전에 봤는가).

## ① KINAC 4대 카테고리 분류

`config/search-config.json`의 `classification` 키워드를 기준으로 한다.

| 카테고리 | 키워드 | 판단 |
|----------|--------|------|
| **지역** | 평산, 강선, 영변, 풍계리 | 북한 핵 관련 지명 언급 |
| **핵물질 생산** | 우라늄, 플루토늄 | 핵분열 물질 생산 관련 |
| **핵주기** | 광산, 정련, 변환, 농축, 제조, 연소, 재처리 | 핵연료 주기 공정 단계 |
| **핵주기시설** | 우라늄 농축시설, 원자로, 방사화학실험시설 | 핵주기 관련 물리 시설 |

판단 원칙:
- 한 기사가 **복수 카테고리**에 해당할 수 있다(예: "영변 원자로 재처리" → 지역+핵주기+핵주기시설).
- 4대 카테고리 중 **하나라도 매칭 → `kinac_relevant: true`** (핵활동 직접 관련, 보고서 우선순위 상향).
- 키워드 필터(북한 핵 등)는 통과했으나 4대 카테고리에 미해당 → `kinac_relevant: false`, `categories: []` (배경 동향으로 보존, 버리지 않음).
- 카테고리 판단이 모호하면 가장 가까운 1개를 부여하고 `category_reason`에 모호함을 명시.

## ② 중복제거 태그

| 태그 | 의미 | 조건 |
|------|------|------|
| `new` | 신규 | 이전 7일 소스에 동일 URL·유사 제목 없음 |
| `reported` | 보고됨 | 이전에 이미 보고된 동일 내용(URL 일치 또는 핵심 키워드 80%+ 일치) |
| `update` | 후속보도 | 동일 사건의 후속 + 유의미한 새 정보(새 수치/성명/결과) |

- 이전 **7일**의 `sources/*/index.json`만 읽는다(개별 items는 읽지 않음). 7일 이전은 무시.
- 첫 실행(이전 index 없음) → 전체 `new`.

## 출력 스키마

### index.json (경량 인덱스)
```json
{
  "date": "YYYY-MM-DD",
  "total": 12,
  "new": 5,
  "reported": 6,
  "update": 1,
  "kinac_relevant_count": 8,
  "by_category": { "지역": 2, "핵물질 생산": 3, "핵주기": 4, "핵주기시설": 2 },
  "items": [
    {
      "id": "src-001",
      "title": "기사 제목",
      "url": "https://...",
      "tag": "new",
      "categories": ["지역", "핵주기시설"],
      "kinac_relevant": true,
      "related_report": null
    }
  ]
}
```

### items/src-XXX.json (개별 상세)
```json
{
  "id": "src-001",
  "title": "기사 제목",
  "url": "https://...",
  "snippet": "기사 요약/발췌",
  "source_name": "매체명",
  "published_at": "YYYY-MM-DD HH:MM",
  "discovered_date": "YYYY-MM-DD",
  "matched_keywords": ["북핵"],
  "categories": ["지역", "핵주기시설"],
  "kinac_relevant": true,
  "category_reason": "영변 원자로 재가동 언급 → 지역(영변)+핵주기시설(원자로)",
  "tag": "new",
  "tag_reason": "이전 7일 소스에 동일/유사 항목 없음",
  "related_report": null,
  "related_item": null
}
```

## 원칙
- `reported` 태그 기사도 index + items에 기록한다(추적 가능성 보존).
- `category_reason`/`tag_reason`은 개별 items 파일에 구체적으로 작성한다.
- `related_report`/`related_item`은 `reported`/`update` 태그에만 기록한다.
