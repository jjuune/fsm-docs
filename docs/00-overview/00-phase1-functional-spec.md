# Phase 1 기능 명세서 (SSoT)

## TL;DR
- **여기만 읽으면 Phase 1 전체가 보입니다.**
- 목표: Admin → TM → Tech 배정 체인 운영 + 현장 수행/증빙 + WorkOrder 표준화
- 비목표: 재고 정밀 추적, 정산, 고급 리포트, AuditLog (→ Phase 2)

---

## 🎯 1. 목표 / 비목표

### 목표 (Phase 1 = 내부 효율 개선 A)
- WorkOrder 기반 설치·AS 업무 프로세스 **표준화**
- 현장 증빙(체크리스트·서명·사진) **디지털화**
- Admin → TM → Tech **3단 배정 체인** 확립
- 팀/기사 관리 **기본 CRUD(CRUD-lite)** 제공

### 비목표 (Phase 2 이후)
- Hard delete (Team/User 모두 비활성화만)
- 고급 성과 리포트 / KPI 대시보드
- PDF 사업자정보 자동삽입 (서식 자동화 고도화)
- AuditLog 전체 노출
- 재고/시리얼 정밀 추적, 정산/결제

---

## 🧑‍🤝‍🧑 2. 조직 / 역할 흐름

```
[Admin (본사)]
    │  팀 생성·관리 / WorkOrder 생성·팀 배정 / 전체 조회
    ▼
[Team Manager (TM, 팀장)]
    │  기사 생성·비활성화 / WorkOrder 기사 배정 / 팀 내 조회
    ▼
[Technician (Tech, 기사)]
       WorkOrder 수행 / 체크리스트·서명·사진 입력 / 완료 제출
```

| 역할 | 코드 | 스코프 | 핵심 권한 |
| :--- | :---: | :---: | :--- |
| 본사 관리자 | ADMIN | ORG | 팀·WO 전체 관리, 통계 |
| 팀 관리자 | TM | TEAM | 기사 관리, 팀 내 WO 배정 |
| 기사 | TECH | SELF | 본인 WO 수행·증빙 |

---

## 🔗 3. 배정 체인 / 상태 모델

### WorkOrder 상태 전이
```
DRAFT
  └─(Admin 팀배정)─▶ TEAM_ASSIGNED
                         └─(TM 기사배정)─▶ TECH_ASSIGNED
                                               └─(Tech 시작)─▶ IN_PROGRESS
                                                                   └─(완료 제출)─▶ COMPLETED
(Admin 취소 가능: DRAFT ~ TECH_ASSIGNED)─▶ CANCELLED
```

| 상태 | 설명 |
| :--- | :--- |
| `DRAFT` | 생성됨, 팀 미배정 |
| `TEAM_ASSIGNED` | 팀 배정 완료, 기사 미배정 |
| `TECH_ASSIGNED` | 기사 배정 완료, 현장 이동 전 |
| `IN_PROGRESS` | 기사 현장 수행 중 |
| `COMPLETED` | 완료 제출·검증 통과 |
| `CANCELLED` | 취소 (사유 필수) |

---

## ✅ 4. 완료 제출 검증 규칙

| 조건 | 필수 여부 | 비고 |
| :--- | :---: | :--- |
| 모든 필수 체크리스트 항목 응답 | ✅ 필수 | PASS / FAIL / NA 중 선택 |
| FAIL 항목의 메모 입력 | ✅ 필수 | FAIL 선택 시 자동 강제 |
| 고객 서명(서명자 성명 포함) | ✅ 필수 | 미입력 시 제출 버튼 비활성 |
| 사진(전/후) 업로드 | ⚠️ 토글 | 기본값 OFF (비필수) |
| 조직/팀 기본정보 | ❌ 해당없음 | **완료 제출 요건 아님 (Phase 1)** |

> 서버는 COMPLETED 전환 시 위 조건을 재검증합니다 (클라이언트 우회 방지).

---

## 💻 5. Phase 1 화면 구성

### 핵심 화면
| 화면 ID | 이름 | 역할 |
| :---: | :--- | :--- |
| M-01 | 기사 작업 목록 | Tech |
| M-02 | 기사 작업 상세 (수행·증빙) | Tech |
| T-01 | 팀 대시보드 | TM |
| T-02 | 팀 작업함 (목록) | TM |
| T-03 | 팀 작업 상세 (배정·관리) | TM |
| A-02 | 본사 작업 목록 | Admin |
| A-04 | 본사 작업 상세 | Admin |

### 관리 화면 (Phase 1 추가)
| 화면 ID | 이름 | 역할 |
| :---: | :--- | :--- |
| A-00 | Admin 대시보드 (팀별 운영지표 MVP) | Admin |
| A-01 | 팀 관리 (생성·수정·비활성화) | Admin |
| T-00 | 팀 대시보드 (기사별 운영지표 MVP) | TM |
| T-04 | 기사 관리 (초대·수정·비활성화) | TM |

### 로그인/홈
| 화면 ID | 이름 | 역할 |
| :---: | :--- | :--- |
| L-01 | 로그인 | All |
| H-01 | My Home (역할별 랜딩 라우팅) | All |

---

## 🗄️ 6. Phase 1 Core 데이터 (ERD 요약)

### 핵심 엔티티
| 엔티티 | 핵심 필드 | 비고 |
| :--- | :--- | :--- |
| **ORGANIZATION** | `legal_name`, `biz_reg_no`, `address` | OrgProfile (1개) |
| **TEAM** | `name`(immutable), `status(ACTIVE/INACTIVE)` | 센터 |
| **USER** | `email`, `role`, `team_id`, `status` | 비활성화만 (삭제 금지) |
| **WORK_ORDER** | `type`, `status`, `assigned_team_id`, `assigned_technician_id` | 핵심 |
| **CHECKLIST_ITEM** | `label`, `value(PASS/FAIL/NA)`, `note` | WO당 10~12개 자동 생성 |
| **SIGNATURE** | `signed_by_name`, `file_id`, `signed_at` | 필수 |
| **FILE_OBJECT** | `storage_key`, `type(PHOTO/SIGN/PDF)` | S3 |

> **OrgProfile:** 단일 레코드. Admin만 편집 가능.
> **Team/User:** Phase 1은 INACTIVE(비활성화)만. Hard delete 금지.

---

## ⚙️ 7. 토글 기본값

| 항목 | 기본값 | 변경 주체 |
| :--- | :---: | :--- |
| 사진 업로드 필수 여부 | **OFF** (비필수) | Admin 설정 |
| 완료 후 증빙 수정 | **금지** | 시스템 고정 |
| Team.name 변경 | **불가** (immutable) | 시스템 고정 |
| User/Team 삭제 | **금지** | 시스템 고정 |
| TM의 TECH 생성 | **허용** (팀 스코프) | RBAC 정책 |

---

## 📋 8. DoD (Definition of Done) — 필드테스트 체크리스트

- [ ] 기사가 M-02에서 체크리스트·서명 제출까지 문제없이 완료했는가?
- [ ] TM이 T-02에서 기사 배정(퀵 배정)을 즉시 수행할 수 있는가?
- [ ] Admin이 A-02에서 전체 작업 흐름을 실시간으로 확인 가능한가?
- [ ] 완료 후 PDF 문서가 조회 가능한 형태로 생성되는가?
- [ ] Admin이 A-01에서 팀을 생성하고 비활성화할 수 있는가?
- [ ] TM이 T-04에서 기사를 초대하고 비활성화할 수 있는가?
- [ ] H-01에서 역할에 따라 올바른 랜딩 화면으로 자동 이동되는가?
- [ ] inactive 팀/유저는 로그인 및 배정에서 차단되었는가?

---

## 연관 문서
- [범위 정의 (In/Out)](03-scope.md)
- [역할과 책임 (RBAC)](02-roles.md)
- [권한 매트릭스](../20-product/00-rbac.md)
- [WorkOrder FSM](../10-domain/01-workorder-fsm.md)
- [ERD + Field Dictionary](../40-data/01-erd.md)

## 변경 이력
- **v0.2:** Phase 1 SSoT 신규 작성 — 목표·역할·상태·화면·데이터·토글·DoD 통합 (2026-02-23)
