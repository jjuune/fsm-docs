# Phase 1 운영툴 문서 (v0.2)

이 저장소는 **공간살균기 설치·AS 내부 운영 디지털화(Phase 1)**의 요구사항/스펙/도메인 규칙을 정리한 문서입니다.

- 문서 버전: **v0.2**
- 최초 생성: 2026-02-20 (v0.1.1)
- 업데이트: 2026-02-23 (v0.2 — SSoT/RBAC/ERD/화면 스펙/API 최소세트 보강)
- 목적: **처음 합류한 개발자도 별도 구두 설명 없이 구현 착수**할 수 있도록 "해야 할 일"을 구조화합니다.

## 📌 먼저 읽을 문서 (SSoT)

> **[👉 Phase 1 기능 명세서 (SSoT)](docs/00-overview/00-phase1-functional-spec.md)**
> 여기만 읽으면 전체 구조가 보입니다.

---

## 빠른 착수 (개발 순서 추천)

1. **도메인 규칙 확정**
   - WorkOrder 상태(FSM) / 전이 / 완료처리 규칙
   - [WorkOrder FSM](docs/10-domain/01-workorder-fsm.md)
2. **인증 / 사용자 관리 구현**
   - L-01 로그인 / H-01 역할별 랜딩 (`GET /me`)
   - A-01 팀 관리 CRUD-lite / T-04 기사 관리 CRUD-lite
   - [로그인/홈 스펙](docs/20-product/00-auth-and-home.md) | [API 최소세트](docs/30-implementation/10-api-minimum-set-phase1.md)
3. **M-02(기사 작업 상세) 구현**
   - 체크리스트/서명/사진/완료 제출 + 서버 검증 + PDF 생성 트리거
4. **배정/조회 흐름 구현**
   - 본사(Admin) 작업 생성/팀 배정 (A-02/A-04)
   - 팀(TeamManager) 기사 배정 (T-01~T-03)
5. **운영 대시보드 (MVP)**
   - A-00 Admin 대시보드 / T-00 팀 대시보드
6. **권한(RBAC) + API 레벨 보호**
   - ORG/TEAM/SELF 스코프 강제, Inactive 차단
7. **데이터 모델(ERD) + Field Dictionary 반영**
   - Organization → Team → User Lifecycle

> 상세 내용은 좌측 내비게이션(또는 mkdocs 네비)을 따라가면 됩니다.
