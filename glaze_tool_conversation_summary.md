# Glaze Batch Calculator — 개발 대화 요약

**작성일**: 2026-05-22  
**파일 위치**: `glaze_tool/glaze_batch_calculator.html`

---

## 프로젝트 개요

도예 작가를 위한 **유약 실험 도구**를 단일 HTML 파일로 제작.  
외부 의존성 없이 Notion `/embed` 로 삽입 가능한 독립 실행형 웹앱.

---

## 최종 구현 기능

### 1. 배치 계산기 (Batch Calculator)
- **Glazy 레시피 붙여넣기 파서**: `name\nnumber` 형식 자동 파싱
- "Total base recipe" 구분선 이하 재료 자동 ADD 처리 (`inAddSection` 플래그)
- 목표 용량(g) 입력 시 각 재료 그람 자동 계산 (`pct / 100 * batch`)
- ADD 체크박스: 착색제 구분, 파란 배경으로 시각화
- 여러 레시피 탭 동시 관리 (탭 추가/삭제)
- 기본 레시피 예시 없이 빈 상태로 시작

### 2. 배치 합산표 (Summary Table)
- 모든 레시피 탭의 재료를 하나의 표로 통합
- 레시피별 컬럼 + 합계(Total) 컬럼
- ADD 재료는 별도 섹션으로 구분 (파란 배경 + ADD 뱃지)
- **Notion 붙여넣기 호환 복사**: `ClipboardItem` API로 `text/html` + `text/plain` 동시 복사 → Notion에 표로 붙여넣어짐

### 3. 착색제 실험 플래너 (Colorant Experiment)
- 착색제 선택 및 농도 조합 표 자동 생성
- `COLORANT_MAP`: Glazy 별칭 → 정규 영문명 정규화
- `COLORANT_KO`: 정규 영문명 → 한국어명 + 화학식 (툴팁)
- 기본 착색제 목록: Iron Oxide, Cobalt Carbonate, Cobalt Oxide 등 9종

### 4. 사용 방법 가이드 (접이식)
- `<details>/<summary>` 태그로 접이식 구현
- 6단계 가이드 + 각 단계별 스크린샷 이미지 (base64 내장)
  - ① Glazy에서 레시피 복사 (Glazy Peach Matte 예시 이미지)
  - ② 붙여넣기 & 자동 파싱
  - ③ ADD 착색제 설정
  - ④ 목표 용량 입력
  - ⑤ 여러 레시피 & 배치 합산표

---

## 주요 기술 결정 사항

| 항목 | 결정 |
|------|------|
| 파일 구조 | 단일 HTML, CSS/JS 인라인 |
| 이미지 삽입 | base64 data URI (외부 파일 불필요) |
| Notion 복사 | `ClipboardItem` API (HTML + 평문 동시) |
| 한국어 폰트 | 시스템 폰트 사용 (-apple-system 계열) |
| 상태 저장 | 없음 (새로고침 시 초기화) |

---

## 파서 핵심 로직

```javascript
const fullNorm = raw
  .replace(/([0-9\.]+)Total/gi, '$1\nTotal')
  .replace(/([a-zA-Z\)])([0-9])/g, '$1\n$2');

const baseTotalRe = /^total\s+base/i;
let inAddSection = false;

// 줄이 baseTotalRe와 일치하면 → inAddSection = true
// 이후 재료는 모두 ADD로 처리
```

---

## 착색제 매핑

```javascript
const COLORANT_MAP = {
  'iron oxide, red': 'Iron Oxide',
  'cobalt oxide, black': 'Cobalt Oxide',
  'chrome oxide green': 'Chrome Oxide',
  // ... 등 약 20개 별칭
};

const COLORANT_KO = {
  'Iron Oxide': '산화철 (Fe₂O₃)',
  'Cobalt Carbonate': '탄산코발트 (CoCO₃)',
  // ... 등 16개 한국어명
};
```

---

## 개발 과정에서 해결한 문제들

### 파일 위치 문제
- **문제**: 처음에 `/sessions/.../mnt/outputs/`(임시 경로)에 저장 → 세션 종료 후 파일 소실
- **해결**: `/sessions/.../mnt/glaze_tool/`(영구 워크스페이스)로 이전, 전체 재빌드

### 퍼센트 vs 그람 계산 오류
- **문제**: ADD 재료의 그람값이 틀리게 계산됨
- **해결**: 모든 재료 동일하게 `(pct / 100) * batch` 공식 적용

### Notion 복사 형식
- **문제**: `writeText()`로 복사 시 Notion에 탭 구분 텍스트로 붙여넣어짐
- **해결**: `ClipboardItem`으로 `text/html` MIME 타입 포함 → Notion이 표로 인식

### 착색제 중복 문제
- **문제**: Cr, Chrome oxide 등 같은 재료가 다른 이름으로 중복 등록
- **해결**: `COLORANT_MAP`으로 모든 별칭을 하나의 정규명으로 정규화

### 모드 전환 오류
- **문제**: `switchMode()` 함수에서 'summary' 모드 처리 누락
- **해결**: 3-mode 완전 구현 (`batch` / `summary` / `exp`)

---

## 가이드 이미지 파일 (워크스페이스에 보관)

| 파일명 | 내용 |
|--------|------|
| `1_glazy_Recipe.png` | Glazy 레시피 페이지 예시 (Peach Matte) |
| `2_paste.png` | 붙여넣기 & 파싱 화면 |
| `3_add.png` | ADD 체크 설정 화면 |
| `4_targetamount.png` | 목표 용량 입력 화면 |
| `5_multiple.png` | 여러 레시피 탭 관리 화면 |
| `6.summary.png` | 배치 합산표 화면 |

---

## 현재 미완료 작업

- 가이드 ① 이미지 크기 조정 (사용자가 px 크기 미결정 상태)

---

## 파일 정보

- **경로**: `glaze_tool/glaze_batch_calculator.html`
- **크기**: 약 312KB (이미지 base64 포함)
- **의존성**: 없음 (완전 독립 실행형)
