# CRUD 매트릭스 (Phase 1)

## TL;DR
- **목적:** 역할별 데이터 접근 권한을 정의하여 보안 홀 방지 및 API 설계 가이드 제공
- **규칙:** C(Create), R(Read), U(Update), D(Delete) / `-`는 권한 없음

---

## 📊 1. 핵심 엔티티별 권한 매트릭스

| 엔티티 (Table) | Admin (본사) | Team Manager (팀) | Technician (기사) | 비고 |
| :--- | :---: | :---: | :---: | :--- |
| **User (사용자)** | CRUD | R | R | Admin만 계정 관리 가능 |
| **Team (조직)** | CRUD | R | - | - |
| **Customer (고객사)** | CRUD | R | R | - |
| **Site (현장)** | CRUD | R | R | - |
| **WorkOrder (작업)** | CRUD | R/U | R/U | TM은 배정(U), 기사는 상태(U) |
| **Checklist (결과)** | R | R | RU | 기사가 현장에서 작성 |
| **File / Signature** | R | R | CR | 기사가 생성(C), 삭제 제한 |
| **AuditLog (로그)** | R | - | - | Admin만 감사 가능 |

---

## 🔐 2. WorkOrder 상태별 수정 권한 (U) 상세

| 작업 상태 | Admin | Team Manager | Technician | 배정 가능 여부 |
| :--- | :---: | :---: | :---: | :--- |
| **DRAFT** | 수정 가능 | - | - | 팀 배정 가능 |
| **TEAM_ASSIGNED** | 수정 가능 | 기사 배정 | - | 기사 배정 가능 |
| **TECH_ASSIGNED** | 수정 가능 | 재배정 가능 | 작업 시작 | - |
| **IN_PROGRESS** | 수정 제한 | 재배정 제한 | 체크리스트 입력 | - |
| **COMPLETED** | **수정 불가** | **수정 불가** | **수정 불가** | 최종 잠금 |

---

## 📌 3. 데이터 조회 범위 (Data Scope)
- **Admin:** 시스템 전체 데이터 조회 가능
- **Team Manager:** 본인 소속 팀(`team_id`)에 할당된 데이터만 조회 가능
- **Technician:** 본인(`technician_id`)에게 배정된 작업 데이터만 조회 가능

---

## 연관 문서
- [권역 및 그룹 권한(RBAC)](../20-product/00-rbac.md)
- [API 명세서](03-api.md)

## 변경 이력
- **v0.1:** 역할 및 상태 기반 CRUD 매트릭스 설계 (2025-02-20)
