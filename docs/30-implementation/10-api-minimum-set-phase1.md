# API 최소세트 (Phase 1)

## TL;DR
- **추가/확장:** Auth, Admin(Team), TM(Technician) 관리 API 최소 세트
- **기존 WorkOrder API는 [API 명세서](03-api.md) 유지**
- **가드 규칙:** ORG/TEAM/SELF 스코프 강제, Inactive 처리 정책

---

## 🔐 1. Auth

| 기능 | Method | Endpoint | Role | 비고 |
| :--- | :---: | :--- | :---: | :--- |
| 로그인 | `POST` | `/auth/login` | ALL | Inactive 계정 → 401 반환 |
| 내 정보 조회 | `GET` | `/me` | ALL (SELF) | role/team_id/status 반환 |

**POST /auth/login 요청:**
```json
{
  "email": "user@example.com",
  "password": "..."
}
```

**GET /me 응답:**
```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "홍길동",
    "role": "TM",
    "team_id": "uuid",
    "team_name": "서울 북부 센터",
    "status": "ACTIVE"
  }
}
```

---

## 🏢 2. Admin — 팀 관리 API

| 기능 | Method | Endpoint | Scope | 비고 |
| :--- | :---: | :--- | :---: | :--- |
| 팀 목록 | `GET` | `/admin/teams` | ORG | 상태 필터 지원 |
| 팀 생성 | `POST` | `/admin/teams` | ORG | `name` immutable |
| 팀 수정 | `PATCH` | `/admin/teams/{id}` | ORG | **`name` 필드 무시** |
| 팀 비활성화 | `PATCH` | `/admin/teams/{id}/deactivate` | ORG | 삭제 금지 |
| 팀 운영지표 | `GET` | `/admin/teams/{id}/stats` | ORG | 상태별 WO 카운트 |
| 전체 팀 요약 | `GET` | `/admin/stats/teams` | ORG | A-00 대시보드용 |

**POST /admin/teams 요청:**
```json
{
  "name": "서울 북부 센터",
  "address": "서울시 ...",
  "contact_phone": "02-...",
  "manager_name": "김관리"
}
```

**PATCH /admin/teams/{id}/deactivate 요청:**
```json
{
  "reason": "센터 통합으로 인한 운영 종료"
}
```

---

## 👷 3. TM — 기사 관리 API

| 기능 | Method | Endpoint | Scope | 비고 |
| :--- | :---: | :--- | :---: | :--- |
| 기사 목록 | `GET` | `/team/technicians` | TEAM | 내 팀 기사만 |
| 기사 초대(생성) | `POST` | `/team/technicians` | TEAM | role=TECH 고정 |
| 기사 정보 수정 | `PATCH` | `/team/technicians/{id}` | TEAM | 연락처/표시명 |
| 기사 비활성화 | `PATCH` | `/team/technicians/{id}/deactivate` | TEAM | 삭제 금지 |
| 기사별 운영지표 | `GET` | `/team/stats/technicians` | TEAM | T-00 대시보드용 |

**POST /team/technicians 요청:**
```json
{
  "name": "박기사",
  "phone": "010-...",
  "email": "tech@example.com",
  "temporary_password": "..."
}
```

**PATCH /team/technicians/{id}/deactivate 요청:**
```json
{
  "reason": "퇴사"
}
```

---

## 🛡️ 4. 가드 규칙

### 4.1 스코프 강제
| 역할 | 허용 범위 | 위반 시 |
| :---: | :--- | :--- |
| ADMIN | ORG (전체) | — |
| TM | TEAM (자기 팀만) | 403 Forbidden |
| TECH | SELF (본인 작업만) | 403 Forbidden |

> TM이 다른 팀의 기사에 `PATCH /team/technicians/{id}` 요청 시 403 반환.

### 4.2 Inactive 처리 정책
| 대상 | 처리 |
| :--- | :--- |
| Inactive 유저 로그인 | 401 반환 ("비활성화된 계정") |
| Inactive 팀에 WO 신규 배정 | 400 반환 ("비활성화된 팀") |
| Inactive 기사에 WO 신규 배정 | 400 반환 ("비활성화된 기사") |
| Inactive 기사의 기존 WO | 이미 배정된 건 유지 (재배정 권장) |

### 4.3 Team.name Immutable 강제
- `PATCH /admin/teams/{id}` 요청 body에 `name` 포함 시 → **서버가 해당 필드를 무시**하고 나머지 필드만 업데이트
- 400 에러가 아닌 **Silent ignore** (클라이언트에서 `name` 편집 UI 자체를 비활성화하여 방지)

---

## 공통 규칙
- **Base URL:** `/api/v1`
- **인증:** `Authorization: Bearer <token>`
- **공통 응답 포맷:**
```json
{
  "ok": true,
  "data": { ... },
  "message": null,
  "error_code": null
}
```

---

## 연관 문서
- [WorkOrder API 명세서](03-api.md)
- [권한 매트릭스(RBAC)](../20-product/00-rbac.md)
- [Org/Team 프로필](../10-domain/05-org-team-profile.md)

## 변경 이력
- **v0.2:** 신규 작성 — Auth/Admin(Team)/TM(Technician) API 최소세트, 가드 규칙 (2026-02-23)
