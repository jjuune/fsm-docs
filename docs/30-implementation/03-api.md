# API 명세서 (Phase 1)

## TL;DR
- **규정:** RESTful API, JSON 기반 통신, Bearer Token 인증
- **핵심:** 권한 검증(본인 배정 확인), 상태 전이 제한(COMPLETED 후 수정 불가)

---

## 🔐 공통 규칙 (Common Rules)
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

## 🛠️ 1. WorkOrder (작업 관리)

| 기능 | Method | Endpoint | 설명 |
| :--- | :---: | :--- | :--- |
| **상세 조회** | `GET` | `/workorders/{id}` | 작업 정보, 체크리스트, 서명 상태 통합 조회 |
| **작업 시작** | `POST` | `/workorders/{id}/start` | 상태 변경: `TECH_ASSIGNED` ➔ `IN_PROGRESS` |
| **완료 제출** | `POST` | `/workorders/{id}/complete` | **서버 최종 검증** 후 상태 변경: `COMPLETED` |
| **취소 처리** | `POST` | `/workorders/{id}/cancel` | (Admin 전용) 사유 입력 필수 |

---

## ✅ 2. Checklist (체크리스트)

### [단건 자동 저장]
- **Endpoint:** `PATCH /workorders/{id}/checklist-items/{itemId}`
- **Request Body:**
```json
{
  "value": "PASS | FAIL | NA",
  "note": "FAIL인 경우 사유 기재 (최소 10자)"
}
```

---

## ✍️ 3. Signature & Attachments (증빙)

| 기능 | Method | Endpoint | 상세 내용 |
| :--- | :---: | :--- | :--- |
| **업로드 URL** | `POST` | `/workorders/{id}/upload-url` | S3 Pre-signed URL 또는 서버 업로드 경로 발급 |
| **서명 등록** | `POST` | `/workorders/{id}/signature` | `signed_by_name`, `file_id`를 작업에 매핑 |
| **사진 등록** | `POST` | `/workorders/{id}/attachments` | `category(BEFORE/AFTER)`, `file_id` 등록 |
| **사진 삭제** | `DELETE` | `/workorders/{id}/attachments/{aid}` | `COMPLETED` 상태 이전만 가능 |

---

## 🏁 4. 시스템 후처리 (Post-process)

### [발송 및 PDF 상태 조회]
- **Endpoint:** `GET /workorders/{id}/delivery-status`
- **응답 예시:**
```json
{
  "pdf": { "status": "READY", "url": "https://..." },
  "sms": { "status": "SENT", "sent_at": "..." },
  "email": { "status": "FAILED", "error": "Invalid address" }
}
```

---

## 📂 연관 문서
- [DB 엔티티 관계(ERD)](../40-data/01-erd.md)
- [권한 매트릭스(RBAC Matrix)](../20-product/00-rbac.md)

## 변경 이력
- **v0.1:** Phase 1 최소 기능 단위 API 명세 (2025-02-20)
