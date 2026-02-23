# WorkOrder FSM

## TL;DR
- Phase 1의 WorkOrder 상태 모델과 전이 규칙을 정의한다.
- 완료(COMPLETED) 시 서버는 **PDF 생성/보관(최소)** 까지만 Phase 1 범위로 포함한다.
- 취소(CANCELLED)는 **사유 필수**, 운영 안전을 위해 **COMPLETED는 취소 불가(권장)** 로 둔다.

---

## 1) WorkOrder 상태 정의

| 상태명 | 설명 | 비고 |
| :--- | :--- | :--- |
| **DRAFT** | 작업 생성 직후(임시 저장 포함) | Admin만 생성/편집 |
| **TEAM_ASSIGNED** | Admin이 팀 배정 완료 | assigned_team_id 확정 |
| **TECH_ASSIGNED** | Team Manager가 기사 배정 완료 | assigned_technician_id 확정 |
| **IN_PROGRESS** | Technician이 '작업 시작' 수행 | 권장 필수 단계 |
| **COMPLETED** | 체크리스트/서명 검증 통과 후 완료 제출 | 이후 수정 기본 불가 |
| **CANCELLED** | 작업 취소 | 사유 필수 |

---

## 2) 상태 전이 규칙 (Flow)

1. **DRAFT**  
   - `Admin: 팀 배정` → **TEAM_ASSIGNED**

2. **TEAM_ASSIGNED**  
   - `Team Manager: 기사 배정` → **TECH_ASSIGNED**

3. **TECH_ASSIGNED**  
   - `Technician: 작업 시작` → **IN_PROGRESS**

4. **IN_PROGRESS**  
   - `Technician: 체크리스트 완료 + 서명 저장 + 완료 제출` → **COMPLETED**

---

## 3) 취소(CANCELLED) 규칙

- **취소 사유는 필수**(reason required).
- 권한(Phase 1 권장):
  - **Admin**: DRAFT / TEAM_ASSIGNED / TECH_ASSIGNED / IN_PROGRESS에서 취소 가능
  - **Team Manager**: (정책 토글로) TEAM 범위 내에서 취소 허용 여부 결정
- **COMPLETED 취소 불가(권장)**: 운영/증빙 무결성 확보를 위해 Phase 1 기본값은 “불가”.

---

## 4) 완료(COMPLETED) 시 서버 자동 처리 (Phase 1)

완료 제출 후 서버는 아래를 순차 처리한다.

- [ ] **데이터 검증:** 필수 체크리스트 입력 여부, FAIL 메모 여부, 서명 데이터 존재 여부
- [ ] **PDF 생성/보관:** 최종 작업 보고서 PDF 생성 및 스토리지 저장 (최소)
- [ ] (선택) **발송:** 문자/이메일 자동 발송은 정책/운영 준비에 따라 단계적 적용

> AuditLog/상세 감사 추적은 Phase 2에서 다룬다.

---

## 5) 화면 목록 및 기능 매핑 (Phase 1)

| 역할 | 화면 ID | 화면명 | 주요 기능 | 연관 데이터 |
| :--- | :---: | :--- | :--- | :--- |
| **공통** | L-01 | 로그인 | 인증, role 기반 라우팅 | User |
| **공통** | H-01 | My Home | Admin/TM/Tech 랜딩 | User, Team |
| **Admin** | A-00 | Admin 대시보드 | 팀별 상태 카운트/지연(운영지표) | WorkOrder, Team |
| **Admin** | A-01 | 팀 관리 | 팀 생성/프로필 수정/비활성 | Team |
| **Admin** | A-02 | 본사 작업 목록 | ORG 범위 조회/필터/팀 배정 | WorkOrder |
| **Admin** | A-04 | 본사 작업 상세 | 상세/팀 변경/취소 | WorkOrder |
| **Team Manager** | T-00 | 팀 대시보드 | 기사별 상태 카운트(운영지표) | WorkOrder, User |
| **Team Manager** | T-04 | 기사 관리 | 기사 생성/수정/비활성 | User |
| **Team Manager** | T-02 | 팀 작업함(목록) | TEAM 범위 조회/기사 배정 | WorkOrder |
| **Team Manager** | T-03 | 팀 작업 상세 | 배정/재배정(정책 토글) | WorkOrder |
| **Technician** | M-01 | 기사 작업 목록 | SELF 범위 목록/진입 | WorkOrder |
| **Technician** | M-02 | 기사 작업 상세 | 체크리스트/서명/사진/제출 | WorkOrder, Signature, Attachment |

---

## 연관 문서
- [화면 지도](../20-product/01-screen-map.md)
- [체크리스트 정책](02-checklist-policy.md)

## 변경 이력
- v0.2: Phase 1 SSoT 기준으로 정렬 (취소/자동처리/화면코드 정합)
