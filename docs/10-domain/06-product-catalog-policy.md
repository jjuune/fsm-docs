# 상품 카탈로그 정책 (Product Catalog Policy)

## TL;DR

- Phase 1 WorkOrder 중심 구조를 유지하면서, **상품 카탈로그(분류/상품/선택)**를 확장한다.
- **가격 정보(price)는 Phase 1 범위 제외** (향후 확장 고려만).
- Admin이 상품 분류/상품을 관리하고, WorkOrder 작성 시 설치 상품을 선택할 수 있다.
- TM/Technician은 기본적으로 **조회 중심(read-only)** 으로 시작한다.

---

## 1. 목표

### 목표 (In)
- 상품 분류(Category) 관리 (CRUD-lite)
- 상품(Product) 관리 (CRUD-lite)
- WorkOrder에 설치/적용 상품 선택
- TM/기사 화면에서 선택된 상품 조회

### 비목표 (Out, Phase 1.x)
- 가격/견적/할인/세금 계산
- 재고/입출고/시리얼 정밀 추적
- 발주/구매 연동
- 상품 옵션 조합(복잡한 Variant)
- 다국어 카탈로그 운영

---

## 2. 핵심 엔티티 개요

- **ProductCategory**: 제품군 분류 (예: 공기살균기, 소모품, 센서모듈)
- **Product**: 실제 등록 상품 (제품명, 사진, 상태 등)
- **WorkOrderProduct**: WorkOrder에 선택된 상품 연결 정보

> 가격 필드는 당장 사용하지 않지만, 향후 확장을 위해 메모 수준으로만 고려할 수 있음 (문서/DB 실제 적용은 Phase 2 검토).

---

## 3. 권한 정책 (RBAC)

### Admin (ORG)
- ProductCategory 생성/수정/비활성화
- Product 생성/수정/비활성화
- WorkOrder 생성/수정 시 상품 선택/해제
- 전체 상품/분류 조회

### Team Manager (TEAM)
- 기본: 상품/분류 조회만
- 담당 팀 WorkOrder 상세에서 선택된 상품 조회

### Technician (SELF)
- 본인 배정 WorkOrder 상세에서 선택된 상품 조회
- 상품/분류 직접 수정 불가

---

## 4. 삭제/비활성 정책 (중요)

### 공통 원칙
- Phase 1.x에서는 **hard delete 지양**
- 기본 정책: `inactive(비활성)` 처리

### ProductCategory
- 하위 활성 상품이 존재하면 삭제(또는 비활성) 시 정책 확인 필요
- 권장:
  - 삭제 대신 비활성화
  - 하위 상품 존재 시 “즉시 삭제 불가” 처리

### Product
- 기존 WorkOrder에 연결된 상품은 hard delete 금지
- 권장:
  - 비활성 처리 후 신규 선택에서만 제외
  - 과거 WorkOrder 기록은 유지

---

## 5. 운영 규칙

- 상품 분류명/상품명은 수정 가능하되, 과거 WorkOrder 이력 해석에 영향이 있으므로 변경 로그 필요 (Phase 2 AuditLog 상세화)
- 상품 비활성 시:
  - 신규 WorkOrder 선택 목록에서는 숨김
  - 기존 WorkOrder에 이미 연결된 항목은 상세 화면에서 계속 표시
- 상품 이미지:
  - Phase 1.x는 **대표 이미지 1장** 기준 권장
  - 이미지 미등록도 허용 가능 (정책 선택)

---

## 6. WorkOrder 연동 정책

### Admin 화면
- WorkOrder 생성/상세에서 상품 선택 가능
- 선택 방식:
  - 카테고리 필터 + 상품 검색 (권장)
  - 초기 MVP는 단순 리스트 선택도 허용

### Team Manager / Technician 화면
- 선택된 상품 조회 가능
- 수정 불가 (Phase 1 기준)

### 저장 시점
- WorkOrder 저장 시 연결 테이블(WorkOrderProduct)에 반영
- 상태 전이(IN_PROGRESS/COMPLETED)와 직접 충돌하지 않도록 설계

---

## 7. 품질/검증 포인트

- 비활성 상품이 신규 WorkOrder 선택 목록에 노출되지 않는지
- 기존 WorkOrder 상세에서 비활성 상품이 정상 표시되는지
- TEAM/SELF 스코프에서 타팀/타기사 WorkOrder 상품정보 접근 차단되는지
- 대량 상품 환경(수십~수백 개)에서 검색/필터 UX가 최소 수준으로 동작하는지

---

## 8. Phase 2 확장 메모

- 가격/단가/통화/유효기간
- 재고/시리얼/LOT
- 상품 옵션(Variant)
- 카탈로그 이력/AuditLog 상세 추적
- 견적/주문/청구 연동