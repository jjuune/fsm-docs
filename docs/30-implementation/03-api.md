# API 초안(dev.Phase1)

## TL;DR
- Phase1에서 필요한 최소 API 세트를 정의하고, 요청/응답 필드를 고정한다.

## 본문
## M-02 기사 화면 API(dev.Phase1)

M-02 기사 화면 API 최소 세트 (Phase 1)
공통 규칙
• 인증: Authorization: Bearer <token> (또는 세션 쿠키)
• 모든 API는 서버에서 권한 검증 필수
o workorder.assigned_technician_id == current_user.id 인지
• 응답은 최소한:
o ok
o error_code
o message
o data

1) WorkOrder 상세 조회 (화면 진입)
1.1 작업 상세 가져오기
GET /api/workorders/{workOrderId}
포함 데이터(기사 화면용 최소)
• WorkOrder: id, type, status, scheduled_start_at, summary, description
• Customer: name
• Site: name, address, contact_name, contact_phone, contact_email
• assigned_team(이름만), assigned_technician(본인)
• notify defaults (phone/email) (있으면)
서버 검증
• 권한(본인 배정)
• status가 접근 가능한 상태인지

1.2 체크리스트 항목 목록
(상세 응답에 같이 포함시키는 것을 권장하지만, 분리도 가능)
GET /api/workorders/{workOrderId}/checklist-items
응답
• items: [{id, section, label, value, note, sort_order, required}]
required는 Phase 1에서 “템플릿 복제 방식”이면 항목에 함께 저장돼 있어야 함.

1.3 첨부/서명 상태 조회
(상세 응답에 같이 포함 권장)
GET /api/workorders/{workOrderId}/attachments
응답
• photos_before: [{file_id, url, created_at}]
• photos_after: [{file_id, url, created_at}]
• pdf: {file_id, url} | null
• signature: {signed_by_name, signed_at, signature_url} | null

2) 체크리스트 자동 저장 (핵심)
2.1 체크리스트 항목 업데이트(단건 autosave)
PATCH /api/workorders/{workOrderId}/checklist-items/{itemId}
body
• value: PASS|FAIL|NA
• note: string | null
응답
• updated item + saved_at
서버 검증
• 권한(본인 배정)
• FAIL이면 note 필수(정책)
• status가 COMPLETED면 수정 불가
대안: 네트워크가 불안하면 “배치 저장” API를 추가할 수도 있음.

2.2 체크리스트 배치 저장(선택)
PATCH /api/workorders/{workOrderId}/checklist-items
body
• items: [{itemId, value, note}]
용도
• 한 번에 저장, 또는 “완료 제출 직전 최종 저장”용

3) 서명 저장 (필수)
3.1 서명 업로드 URL 발급 (권장: direct upload)
POST /api/workorders/{workOrderId}/signature/upload-url
응답
• upload_url (pre-signed)
• file_id (예정)
• headers(optional)
구현 단순화를 원하면 서버에 multipart 업로드로 받아도 됨.
하지만 파일이 커질 수 있는 사진까지 고려하면 “직접 업로드” 패턴이 안정적.

3.2 서명 메타 저장(서명 등록)
POST /api/workorders/{workOrderId}/signature
body
• signed_by_name (required)
• signed_by_role (optional)
• file_id (signature image file id)
응답
• signature: {signed_by_name, signed_at, signature_url}
서버 검증
• 서명 파일 존재 여부
• signer name 필수
• COMPLETED면 변경 불가(또는 “다시 받기” 허용 정책 시 PUT 지원)

3.3 서명 삭제/재입력(선택)
DELETE /api/workorders/{workOrderId}/signature
필드테스트에선 굳이 없어도 됨(그냥 overwrite 허용).

4) 사진 업로드(권장)
4.1 사진 업로드 URL 발급
POST /api/workorders/{workOrderId}/photos/upload-url
body
• category: BEFORE_PHOTO|AFTER_PHOTO
응답
• upload_url
• file_id

4.2 사진 첨부 등록(Attachment 생성)
POST /api/workorders/{workOrderId}/attachments
body
• file_id
• category: BEFORE_PHOTO|AFTER_PHOTO
응답
• attachment record

4.3 사진 삭제(선택)
DELETE /api/workorders/{workOrderId}/attachments/{attachmentId}
정책
• COMPLETED 이후 삭제 불가(권장)

5) 고객 발송 정보(선택)
5.1 발송 대상 정보 저장(오더 필드 업데이트)
PATCH /api/workorders/{workOrderId}
body (기사 허용 범위만)
• customer_notify_phone (optional)
• customer_notify_email (optional)
• notify_sms_enabled (boolean)
• notify_email_enabled (boolean)
서버 검증
• 전화/이메일 형식 검증(옵션)
• COMPLETED 이후 변경 제한(권장)

6) 작업 시작(선택)
6.1 작업 시작
POST /api/workorders/{workOrderId}/start
효과
• status: TECH_ASSIGNED → IN_PROGRESS
• started_at 기록(선택)
• AuditLog 기록

7) 완료 제출(가장 중요)
7.1 완료 제출
POST /api/workorders/{workOrderId}/complete
body (선택)
• finalize_checklist: true/false (배치 저장을 같이 할지)
• notes(optional): “기사 메모” (있으면)
서버 검증(필수)
1. 필수 체크리스트 전부 value 존재
2. FAIL 항목 note 존재
3. 서명 존재 + 서명자 이름 존재
4. (정책) 사진 최소 조건
5. status가 COMPLETED/CANCELLED면 거부
성공 응답
• workorder: {id, status=COMPLETED, completed_at}
• pdf_status: QUEUED|GENERATING|READY (선택)
• notify_status: QUEUED|SENT|FAILED (선택)
서버 후처리(비동기 권장)
• PDF 생성 잡 enqueue
• 문자/이메일 발송 잡 enqueue
• AuditLog 기록(성공/실패)

8) 발송/문서 상태 조회(완료 후 화면용)
8.1 PDF/발송 상태 조회
GET /api/workorders/{workOrderId}/delivery-status
응답
• pdf: {status, url|null, generated_at|null}
• sms: {enabled, status, sent_at|null, error|null}
• email: {enabled, status, sent_at|null, error|null}
기사 화면에서는 “완료 후 자동 처리 중입니다”만 보여줘도 되고,
관리자 화면에서만 상세를 보여줘도 됨.

(선택) 에러 코드 규약 예시
• ERR_UNAUTHORIZED
• ERR_FORBIDDEN_NOT_ASSIGNED
• ERR_INVALID_STATUS
• ERR_CHECKLIST_INCOMPLETE
• ERR_SIGNATURE_REQUIRED
• ERR_VALIDATION_EMAIL
• ERR_VALIDATION_PHONE
• ERR_UPLOAD_FAILED
• ERR_SERVER

권장 최소 구현 세트(진짜 최소)
필드테스트를 돌아가게만 하려면 아래 6개만 있어도 돼:
1. GET workorder detail (+ checklist + signature status 포함)
2. PATCH checklist item
3. POST signature upload-url (또는 직접 업로드)
4. POST signature register
5. POST complete
6. GET delivery-status (또는 workorder detail에 포함)

(본문 끝)
===

## 연관 문서
- (추가 예정)

## 변경 이력
- v0.1: 통합문서 기반 초안 정리
