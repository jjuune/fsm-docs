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

## 📘 5. 권한 가드 예시 (필수, Phase 1)

서버는 모든 API에서 RBAC를 **반드시** 강제한다. (UI에서 숨겨도 서버가 최종 방어)

### 5.1 원칙
- **Admin = ORG**: 조직 전체 데이터 접근 가능
- **Team Manager = TEAM**: `assigned_team_id = my_team_id` 범위만
- **Technician = SELF**: `assigned_technician_id = me` 범위만

### 5.2 예시 1) Technician 내 작업 목록: SELF 강제
`GET /me/workorders`

- 서버는 토큰의 `user_id`를 기준으로 **반드시** 아래 조건을 적용한다.
  - `workorder.assigned_technician_id = me`
- 클라이언트가 임의로 `assigned_technician_id`를 넘기거나 조작해도 무시한다.

### 5.3 예시 2) Team Manager 기사 목록/관리: TEAM 강제
`GET /team/technicians`
`POST /team/technicians`
`PATCH /team/technicians/{id}`

- 조회/수정 대상 `User.team_id`가 **반드시** `my_team_id`여야 한다.
- Team Manager는 **다른 팀** 기사 정보를 조회/수정/비활성화할 수 없다.

### 5.4 예시 3) Team Manager 팀 작업함: TEAM 강제
`GET /team/workorders`

- 조회 조건: `workorder.assigned_team_id = my_team_id`
- (정책 토글) 기사 재배정이 허용된 경우에도 TEAM 범위를 벗어난 배정은 불가

### 5.5 예시 4) Admin 팀 관리/통계: ORG 강제
`GET /admin/teams`
`POST /admin/teams`
`GET /admin/teams/{id}/stats`

- Admin은 ORG 범위 접근 가능
- 다만 팀 비활성화/정책 변경은 **Audit 대상(Phase 2)** 로 분리 가능

### 5.6 예시 5) 비활성 계정/팀 처리 (권장)
- `User.status = INACTIVE` 이면 로그인/토큰 갱신을 차단한다.
- `Team.status = INACTIVE` 인 팀의 신규 배정/진행 전이는 정책에 따라 차단한다.

---

## Product Catalog 확장 API 최소 세트 (Gate 2 후보 / Phase 1.x)

> 가격/재고 제외. 분류/상품 관리 + WorkOrder 상품 선택/조회 중심.


### 1) ProductCategory API (Admin)


#### GET /product-categories

- 목적: 분류 목록 조회 (Admin 관리 화면 / WorkOrder 상품선택 필터)
- 권한: Admin (ORG), 필요 시 TM 조회 허용 가능
- query (예시):
  - is_active=true|false
  - q=검색어
  - page, page_size


#### POST /product-categories
- 목적: 분류 생성
- 권한: Admin
- body (예시): name, code, sort_order

```  JSON
{
"name": "공기살균기",
"code": "STERILIZER",
"sort_order": 10
}
```

#### PATCH /product-categories/{categoryId}
- 목적: 분류 수정/비활성
- 권한: Admin
- body (예시): name, is_active

``` JSON
{
"name": "공기살균정화기",
"is_active": true
}
```

#### DELETE /product-categories/{categoryId} (선택)
- 목적: 삭제 시도 (실제로는 비활성 처리 권장)
- 권한: Admin
- 정책:
   - 하위 활성 상품 존재 시 삭제 거부(409) 권장


### 2) Product API (Admin 중심)

#### GET /products
- 목적: 상품 목록 조회
- 권한: Admin / (TM 조회 허용 가능)
- query (예시):
   - category_id
   - is_active=true|false
   - q=검색어
   - page, page_size


#### POST /products
- 목적: 상품 생성
- 권한: Admin
- body (예시): category_id, name, model_name, sku, description, primary_image_file_id

``` JSON 
{
"category_id": "cat_001",
"name": "OH 라디컬 공기살균기 A1",
"model_name": "A1",
"sku": "OH-A1",
"description": "기본형 모델",
"primary_image_file_id": "file_123"
}
```

#### PATCH /products/{productId}
- 목적: 상품 수정/비활성
- 권한: Admin
- body (예시): name, is_active

``` JSON
{
"name": "OH 라디컬 공기살균기 A1 (개정)",
"is_active": true
}
```

#### DELETE /products/{productId} (선택)
- 목적: 삭제 시도
- 권한: Admin
- 정책:
   - WorkOrder 연결 이력 존재 시 삭제 거부(409) 권장
   - Phase 1.x는 비활성 처리 우선


### 3) WorkOrder 상품 선택/조회 API

#### GET /workorders/{workOrderId}/products
- 목적: 해당 WorkOrder에 연결된 상품 목록 조회
- 권한:
   - Admin (ORG)
   - TM (TEAM scope)
   - Technician (SELF scope, 본인 배정 건)

- 응답 예시

``` JSON
{
"items": [
{
"id": "wop_001",
"product_id": "prod_101",
"product_name": "OH 라디컬 공기살균기 A1",
"category_name": "공기살균기",
"quantity": 1,
"note": "거실 벽면 설치"
}
]
}
```

#### PUT /workorders/{workOrderId}/products
- 목적: WorkOrder 연결 상품 전체 교체 (간단한 MVP 방식)
- 권한: Admin (Phase 1.x 기본)
- body 예시

```JSON
{
"items": [
{
"product_id": "prod_101",
"quantity": 1,
"note": "거실 벽면 설치"
},
{
"product_id": "prod_205",
"quantity": 2,
"note": "필터 예비분"
}
]
}
```

- 서버 검증:
   - 비활성 상품 선택 불가
   - 중복 product_id 처리 정책 정의 (합치기 또는 400)
   - WorkOrder 상태별 수정 허용 범위 정책 확인

MVP에선 PUT 전체교체가 구현 단순성 측면에서 유리.
추후 POST/PATCH/DELETE /workorder-products/{id} 세분화 가능.


### 4) 이미지 업로드 연계 (선택)

#### POST /files (기존 파일 업로드 API 재사용)
- 목적: 상품 대표 이미지 업로드
- 권한: Admin
- 결과: FileObject.id 반환 → Product 생성/수정 시 참조


### 5) 오류 코드/정책 예시

- 403 FORBIDDEN
   - RBAC 스코프 위반 (TM/Technician의 상품 수정 시도)
- 404 NOT_FOUND
   - 존재하지 않는 category/product/workorder
- 409 CONFLICT
   - 분류 삭제 불가(하위 활성 상품 존재)
   - 상품 삭제 불가(WorkOrder 연결 이력 존재)
   - 상태 정책상 WorkOrder 상품 수정 불가
- 422 UNPROCESSABLE_ENTITY
   - 필수값 누락 / invalid quantity / 비활성 상품 선택 시도


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
