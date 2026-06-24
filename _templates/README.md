---
title: "_templates 사용법"
type: meta
tags: [meta, 템플릿]
---

← [[00_Home]]

# _templates 사용법

이 폴더는 **코어 Templates 플러그인**용 빈 골격 모음입니다. 손으로 frontmatter를 베끼지 말고 여기서 삽입하세요.

## 최초 1회 설정
1. `설정 → 코어 플러그인 → 템플릿(Templates)` 켜기
2. `설정 → 템플릿 → 템플릿 폴더 위치` → **`_templates`** 지정
3. (선택) `설정 → 단축키 → "템플릿 삽입"`에 키 할당

## 사용 흐름
1. 새 노트를 **올바른 폴더**에 생성 (개념→`02_Concepts`, 논문→`01_Papers` …)
2. 파일명 = 노트 제목 (`{{title}}`가 이 이름으로 치환됨)
3. `Ctrl+P` → "템플릿 삽입" → 해당 템플릿 선택

## 템플릿 목록
| 템플릿 | 대상 폴더 | type |
|--------|----------|------|
| `Concept.md` | `02_Concepts/` | concept |
| `Paper.md` | `01_Papers/` | paper |
| `Daily.md` | `99_Daily/` | daily |
| `MOC.md` | `00_MOC/` | MOC |
| `Project.md` | `03_Projects/` | project |

## 치환 변수
- `{{title}}` — 파일명
- `{{date}}` / `{{date:YYYY-MM-DD}}`
- `{{time}}`

## 삽입 후 채울 것
`tags` / `aliases` / `authors·year·venue`(논문) · 역링크 헤더의 `MOC_AVP-ROS` · `관련 노트`의 `[[]]`.

## Templater로 전환 시 (고급)
폴더별 자동 분기·고급 변수가 필요하면 **Templater** 커뮤니티 플러그인으로:
- `{{title}}` → `<% tp.file.title %>`
- `{{date:YYYY-MM-DD}}` → `<% tp.date.now("YYYY-MM-DD") %>`
