# M-01 기사 작업 목록 (모바일 수행 화면)

## TL;DR
- **목적:** 기사가 로그인 직후 “내가 오늘/진행 중인 작업”을 즉시 확인하고, **M-02(작업 상세)**로 빠르게 진입한다.
- **스코프(RBAC):** TECH = SELF 범위(assigned_technician_id = me)
- **핵심 KPI(운영):** “M-01에서 M-02 진입까지” 평균 10초 이내, 당일 작업 식별 실수 최소화

---

## 1) 화면 구성 (최소 스펙)

### 상단 헤더
- 타이틀: `내 작업`
- 보조: 새로고침(아이콘) / 로그아웃(옵션, Phase 1에서는 메뉴로 대체 가능)

### 탭(권장, Phase 1 MVP)
- `진행중` / `오늘` / `전체`
  - **진행중:** status = IN_PROGRESS
  - **오늘:** scheduled_at = today AND status in (TECH_ASSIGNED, IN_PROGRESS)
  - **전체:** status in (TECH_ASSIGNED, IN_PROGRESS) + (옵션) 최근 COMPLETED 일부 노출

> 탭이 부담이면, Phase 1에서는 “진행중 우선 + 나머지 날짜순” 1개 목록으로 시작해도 됨.

### 리스트(카드) 필드
각 WorkOrder 카드에 아래 필드를 표시한다.

- 상태 배지: `진행중` / `배정됨` / (옵션) `완료`
- 작업 타입: INSTALL / AS
- 현장/고객 요약: `현장명 또는 고객명`
- 주소(1줄): `시/군/구 + 도로명 일부`(너무 길면 ellipsis)
- 일정: `오늘 14:00` / `미정`
- CTA: `열기` (M-02로 이동)

### 카드 탭 시 동작
- 기본: 카드 전체 탭 = M-02 진입
- (선택) 전화/지도는 M-02 상단 액션으로 제공 (M-01에서는 최소화)

---

## 2) 정렬/필터 규칙 (Phase 1 기준)

### 정렬 우선순위
1. IN_PROGRESS (가장 위)
2. TECH_ASSIGNED 중 scheduled_at이 오늘인 것
3. TECH_ASSIGNED 중 scheduled_at이 빠른 순
4. scheduled_at 없는 건 맨 아래

### 지연(Overdue) 표시 (권장)
- 조건: scheduled_at < now AND status != COMPLETED/CANCELLED
- 배지: `지연` (빨간색/강조는 UI 단계에서)

---

## 3) 빈 상태/오류 상태

### 빈 상태
- 문구: `배정된 작업이 없습니다.`
- 보조: `팀 관리자에게 작업 배정을 요청하세요.`

### 네트워크 오류
- 토스트: `네트워크가 불안정합니다. 다시 시도해주세요.`
- 버튼: `재시도`

---

## 4) 전환 규칙 (M-01 → M-02)

- M-01에서 WorkOrder를 탭하면 M-02로 이동
- M-02에서 완료 제출(COMPLETED) 성공 시:
  - 기본: M-01로 돌아오고 해당 작업은 목록에서 제거되거나 “완료”로 이동(정책 선택)
- 세션 만료 시:
  - L-01 로그인으로 이동(과투자 방지: H-01은 라우팅 레이어)

---

## 5) 최소 API 세트 (Phase 1)

### GET 내 작업 목록
`GET /me/workorders`

#### Query (권장)
- `status`: TECH_ASSIGNED | IN_PROGRESS | COMPLETED (옵션)
- `date`: YYYY-MM-DD (오늘 탭)
- `include_recent_completed`: true/false (옵션)

#### Response (필드 최소)
- `id`
- `type` (INSTALL|AS)
- `status`
- `scheduled_at` (nullable)
- `customer_name`
- `site_name`
- `site_address_short`
- `contact_phone` (옵션)

### GET 작업 상세
- M-02에서 사용: `GET /workorders/{id}`

---

## 6) 구현 리스크/주의사항 (짧게)

- **목록 성능:** 페이지네이션(Phase 1은 limit 20으로 충분)
- **정합성:** 서버는 SELF 스코프 강제(assigned_technician_id = me)
- **오프라인:** Phase 1은 “네트워크 불안정 대응(재시도/토스트)”까지만