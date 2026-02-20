# 데이터 모델(ERD)

## TL;DR
- ERD v0.1(논리 모델)과 lite 버전을 함께 제공하며, WorkOrder 필드 사전을 기준으로 구현합니다.

## 본문
ERD v0.1 (Phase 1) — 논리 데이터 모델
1) 핵심 엔티티 목록
• User (Admin / TeamManager / Technician)
• Team
• Customer
• Site (현장)
• WorkOrder (설치/AS 작업)
• ChecklistTemplate / ChecklistTemplateItem (템플릿)
• WorkOrderChecklistItem (오더별 체크리스트 실제 값)
• FileObject (업로드 파일 메타: 사진/서명/PDF)
• WorkOrderAttachment (오더-파일 연결)
• WorkOrderSignature (서명 메타)
• DeliveryLog (문자/이메일 발송 상태)
• AuditLog (행동 이력)
Phase 1에서 “재고/시리얼 추적”을 진짜로 하려면 ProductUnit이 들어오는데, 지금 목표(A 내부 효율)에서는 WorkOrder에 serial_text로만 우선 넣고, Phase 2에서 ProductUnit으로 승격하는 게 부담이 적어.

2) 테이블별 필드(권장 최소)
2.1 Team
• id (PK)
• name
• region_code (optional)
• is_active
• created_at, updated_at
2.2 User
• id (PK)
• name
• email (unique)
• phone (optional)
• role ENUM(ADMIN,TEAM_MANAGER,TECHNICIAN)
• team_id (FK Team, nullable for ADMIN)
• is_active
• created_at, updated_at
2.3 Customer
• id
• name
• contact_name (optional)
• contact_phone (optional)
• contact_email (optional)
• memo (optional)
• created_at, updated_at
2.4 Site
• id
• customer_id (FK Customer)
• name
• address_line1
• address_line2 (optional)
• contact_name (optional)
• contact_phone (optional)
• contact_email (optional)
• memo (optional)
• created_at, updated_at
2.5 WorkOrder
• id
• type ENUM(INSTALL,SERVICE)
• status ENUM(DRAFT,TEAM_ASSIGNED,TECH_ASSIGNED,IN_PROGRESS,COMPLETED,CANCELLED)
• customer_id (FK Customer)
• site_id (FK Site)
• summary (짧은 요약)
• description (특이사항/상세)
• scheduled_start_at (optional)
• priority ENUM(LOW,NORMAL,HIGH) default NORMAL
• 배정:
o assigned_team_id (FK Team, nullable)
o assigned_technician_id (FK User, nullable)
• 완료/취소:
o started_at (optional)
o completed_at (optional)
o cancelled_at (optional)
o cancel_reason (optional)
• 발송 옵션(선택):
o notify_sms_enabled (bool)
o notify_email_enabled (bool)
o customer_notify_phone (optional)
o customer_notify_email (optional)
• “시리얼 텍스트”(Phase 1 가볍게):
o product_serial_text (optional)
• created_by_user_id (FK User)
• created_at, updated_at
상태/배정/발송 옵션은 WorkOrder 한 곳에 모아두면 화면/쿼리가 단순해져.

2.6 ChecklistTemplate / ChecklistTemplateItem (템플릿)
ChecklistTemplate
• id
• type ENUM(INSTALL,SERVICE)
• version (int) — 나중에 항목 바뀌어도 과거 기록 유지
• is_active
• created_at
ChecklistTemplateItem
• id
• template_id (FK ChecklistTemplate)
• section (예: “필수 체크”)
• label (항목명)
• required (bool)
• sort_order (int)
Phase 1은 “현실 버전 필수 10~12개”만 있으니 section은 사실상 1개여도 됨. 그래도 확장 대비로 유지 추천.

2.7 WorkOrderChecklistItem (오더별 실제 입력값)
• id
• work_order_id (FK WorkOrder)
• template_item_id (FK ChecklistTemplateItem) (또는 label 복사 저장 방식)
• value ENUM(PASS,FAIL,NA) nullable
• note (optional)
• updated_by_user_id (FK User)
• updated_at
중요 제약
• (work_order_id, template_item_id) unique

2.8 FileObject (파일 메타)
• id
• type ENUM(PHOTO,SIGNATURE,PDF)
• storage_key (S3/R2 키 같은 것)
• filename (optional)
• content_type
• size_bytes
• created_by_user_id (FK User)
• created_at

2.9 WorkOrderAttachment (오더-파일 연결)
• id
• work_order_id (FK WorkOrder)
• file_id (FK FileObject)
• category ENUM(BEFORE_PHOTO,AFTER_PHOTO,REPORT_PDF,OTHER)
• created_at
제약
• PDF는 오더당 1개만 유지하려면 (work_order_id, category) unique (category=REPORT_PDF)

2.10 WorkOrderSignature (서명 메타)
• id
• work_order_id (FK WorkOrder) unique
• signature_file_id (FK FileObject)
• signed_by_name
• signed_by_role (optional)
• signed_at

2.11 DeliveryLog (발송 상태)
• id
• work_order_id (FK WorkOrder)
• channel ENUM(SMS,EMAIL)
• to_address (phone or email)
• status ENUM(QUEUED,SENT,FAILED)
• provider_message_id (optional)
• error_message (optional)
• attempt_count (int)
• last_attempt_at (optional)
• created_at, updated_at
WorkOrder에 “설정”을 두고, DeliveryLog는 “결과/이력”을 남기는 역할.

2.12 AuditLog (감사/이력)
• id
• work_order_id (FK WorkOrder, nullable — 사용자/팀만 변경도 기록하려면 null 허용)
• actor_user_id (FK User)
• action (예: CREATE_WORKORDER, ASSIGN_TEAM, ASSIGN_TECH, UPDATE_STATUS, UPLOAD_FILE, COMPLETE, RESEND_SMS)
• meta_json (optional: before/after, 사유 등)
• created_at

3) 관계(카디널리티) 요약
• Customer 1 — N Site
• Customer 1 — N WorkOrder
• Site 1 — N WorkOrder
• Team 1 — N User
• Team 1 — N WorkOrder (assigned_team_id)
• User(Technician) 1 — N WorkOrder (assigned_technician_id)
• WorkOrder 1 — N WorkOrderChecklistItem
• WorkOrder 1 — 1 WorkOrderSignature
• WorkOrder 1 — N WorkOrderAttachment
• WorkOrder 1 — N DeliveryLog
• WorkOrder 1 — N AuditLog

4) 인덱스/성능 “최소” 추천
• WorkOrder:
o (assigned_team_id, status, scheduled_start_at)
o (assigned_technician_id, status, scheduled_start_at)
o (status, scheduled_start_at)
o (customer_id) , (site_id)
• WorkOrderChecklistItem:
o (work_order_id)
• WorkOrderAttachment:
o (work_order_id, category)
• DeliveryLog:
o (work_order_id, channel, status)

5) Phase 1에서 “미루는 게 좋은 것”
• ProductModel / ProductUnit / 재고 테이블 (Phase 2)
• 외부 파트너(테넌트) 구조 (Phase 3)
• 권한을 세분화한 Permission 테이블 (지금은 role 기반으로 충분)

다음: 마일스톤(필드테스트 목표 포함) 초안
목표
• 필드테스트 가능한 MVP: “작업 생성 → 팀 배정 → 기사 배정 → 체크리스트/서명 → 완료 제출 → PDF 생성/보관(발송은 옵션)”까지
마일스톤 (주차 단위 예시)
M0: 준비 (2~3일)
• ERD v0.1 확정
• 화면 범위 확정(M-02, T-02, T-03, A-02, A-04)
• 체크리스트 템플릿 v0.2 확정(이미 됨)
M1: 코어 백엔드 + 인증 (1주)
• User/Team/Role
• WorkOrder CRUD + 상태/배정
• AuditLog 기본 기록
M2: 기사 화면 핵심 (1주)
• M-02 구현
• Checklist autosave
• Signature 저장
• 완료 제출(서버 검증)
M3: 팀 운영(1주)
• T-01 대시보드(요약 API)
• T-02 작업함 + 퀵 기사배정
• T-03 작업 상세(증빙 상태 확인)
M4: 본사 운영(1주)
• A-02 목록 + 퀵 팀배정
• A-04 상세(취소/재발송은 옵션)
• PDF 생성/저장(최소)
M5: 필드테스트(1~2주, 병행)
• 실제 기사 1~2명 + 팀장 1명으로 운용
• 이슈 수집(누락/불편/네트워크/서명 UX)
• 체크리스트 필수 항목 조정(12→10 등)
발송(SMS/Email)은 Phase 1에서도 가능하지만, “필드테스트 빨리”가 목표면 PDF까지 먼저 안정화시키고 발송은 뒤에 붙이는 게 리스크가 적어.

(본문 끝)
===

ERD 라이트 v0.1 (Phase 1 Core)
0) 목적
• “작업 생성 → 팀 배정 → 기사 배정 → 체크리스트/서명 → 완료 → PDF 저장(옵션: 발송)”
이 흐름만 빠르게 굴러가게 하는 최소 데이터 구조.

1) 핵심 테이블(9개)
1) Team
• id, name, is_active, created_at
2) User
• id, name, email(unique), phone(optional)
• role: ADMIN / TEAM_MANAGER / TECHNICIAN
• team_id(nullable for ADMIN)
• is_active, created_at
3) Customer
• id, name
• contact_name/phone/email(optional)
• memo(optional), created_at
4) Site
• id, customer_id(FK)
• name, address_line1, address_line2(optional)
• contact_name/phone/email(optional)
• memo(optional), created_at
5) WorkOrder (가장 중요)
• id
• type: INSTALL / SERVICE
• status: DRAFT / TEAM_ASSIGNED / TECH_ASSIGNED / IN_PROGRESS / COMPLETED / CANCELLED
• customer_id(FK), site_id(FK)
• summary, description(optional)
• scheduled_start_at(optional)
• priority(optional: LOW/NORMAL/HIGH)
• assigned_team_id(FK Team, nullable)
• assigned_technician_id(FK User, nullable)
• product_serial_text(optional) ← Phase 1은 텍스트로만
• completed_at(optional), cancel_reason(optional)
• notify_sms_enabled(optional), notify_email_enabled(optional)
• customer_notify_phone/email(optional)
• created_by_user_id(FK User)
• created_at, updated_at
6) WorkOrderChecklistItem (기사 입력값)
• id
• work_order_id(FK)
• label (항목명 텍스트) ← 템플릿 테이블 없이도 운영 가능
• required(bool)
• sort_order(int)
• value: PASS/FAIL/NA (nullable)
• note(optional)
• updated_by_user_id(FK User)
• updated_at
핵심 단순화 포인트: 템플릿/버전 관리는 지금 보류.
“오더 생성 시 체크리스트 항목 10~12개를 복제 생성” 방식으로 충분.
7) FileObject (업로드 파일 메타)
• id
• type: PHOTO / SIGNATURE / PDF
• storage_key (클라우드 스토리지 경로/키)
• content_type, size_bytes
• created_by_user_id(FK User)
• created_at
8) WorkOrderAttachment (오더-파일 연결)
• id
• work_order_id(FK)
• file_id(FK FileObject)
• category: BEFORE_PHOTO / AFTER_PHOTO / REPORT_PDF / OTHER
• created_at
9) WorkOrderSignature (서명 메타)
• id
• work_order_id(FK) unique
• signature_file_id(FK FileObject)
• signed_by_name
• signed_by_role(optional)
• signed_at

2) 관계 요약(한 줄)
• Team 1—N User
• Customer 1—N Site
• Customer 1—N WorkOrder
• Site 1—N WorkOrder
• Team 1—N WorkOrder(assigned_team_id)
• User(Technician) 1—N WorkOrder(assigned_technician_id)
• WorkOrder 1—N ChecklistItem
• WorkOrder 1—1 Signature
• WorkOrder 1—N Attachment → FileObject

3) “나중에 추가”로 미루는 것(명시)
• ChecklistTemplate/버전관리(항목 바뀌기 시작하면 추가)
• DeliveryLog(발송 이력/실패 분석이 필요해지면 추가)
• AuditLog(분쟁/감사 요구가 커지면 추가)
• ProductUnit/재고(진짜 입출고 관리 들어갈 때 추가)
이렇게 문서에 적어두면 “왜 없냐” 질문이 나왔을 때 설명이 깔끔해져.

마일스톤 4단계 (필드테스트까지)
M1) 코어 백엔드 + 본사 최소(팀 배정까지)
목표: WorkOrder 생성/조회 + 팀 배정이 돌아간다
• 인증/역할(ADMIN/TM/T)
• WorkOrder CRUD + 상태/팀 배정
• Customer/Site 최소 등록(또는 WorkOrder 생성 시 함께)
• A-02 목록(팀 미배정 탭 + 퀵 팀 배정)
• A-04 상세(팀 배정/기본 정보 확인 정도)
완료 정의
• Admin이 작업 생성 → 팀 배정 가능
• 팀관리자 계정으로 들어가면 팀 작업이 보임

M2) 기사 화면 핵심(M-02) 완성
목표: 현장에서 “증빙까지” 끝낼 수 있다
• 체크리스트 10~12개(오더 생성 시 자동 생성)
• autosave(PASS/FAIL/NA + FAIL 메모)
• 서명 저장(필수)
• 완료 제출(서버 검증)
• PDF는 “저장만” 또는 “생성 트리거만” (초기엔 단순)
완료 정의
• 기사 1명이 모바일로 체크리스트+서명 → 완료 제출 성공

M3) 팀 운영(T-01/T-02/T-03) 최소
목표: 팀관리자가 “미배정→기사배정→진행 확인” 가능
• T-02 작업함(미배정 탭 기본 + 목록 퀵 배정)
• T-03 작업 상세(기사 배정/재배정, 증빙 상태 확인)
• T-01 대시보드(KPI + 기사별 현황) 간단 버전
완료 정의
• TeamManager가 미배정 작업을 목록에서 기사 배정 가능
• 진행/완료 상태 확인 가능

M4) 필드테스트 1차(실사용 피드백 반영)
목표: “실제 운영에서 막히는 지점”을 찾고 고친다
• 서명 UX/저장 안정성
• 체크리스트 필수 항목 수 조정(12→10 등)
• 사진 업로드는 권장에서 시작(필요하면 정책 토글)
• PDF(저장/다운로드) 안정화
• (선택) 고객 발송은 나중에 붙이거나, 내부 테스트용으로만
완료 정의
• 실제 기사/팀장 1~2명으로 1주 운영해도 큰 사고 없이 돌아감

지금 시점에서 “딱 좋은” 산출물 체크리스트
• ✅ 화면 스펙(M-02/T-01/T-02/T-03/A-02/A-04)
• ✅ 권한 매트릭스 + API 가드
• ✅ ERD 라이트 v0.1
• ✅ 마일스톤 4단계

WorkOrder 생성 시 체크리스트 자동 생성 규칙 (Phase 1)
1) 생성 트리거
• WorkOrder가 최초 생성될 때 체크리스트 항목을 자동 생성한다.
• 생성 시점은 아래 중 하나로 선택(둘 다 가능):
1. POST /workorders 성공 직후(서버 트랜잭션 내부 권장)
2. WorkOrder가 DRAFT로 저장된 직후(동일)
2) 항목 소스
• WorkOrder.type에 따라 고정된 “현실 버전” 항목 세트를 사용한다.
o INSTALL: 필수 12개
o SERVICE(AS): 필수 10개
• Phase 1에서는 템플릿 테이블 없이, 서버 코드에 상수/시드 데이터로 보관해도 된다.
3) 생성 방식
• WorkOrderChecklistItem을 N개 생성한다.
• 각 항목은 최소 필드를 가진다:
o work_order_id
o label
o required=true
o sort_order (1..N)
o value=null (미입력)
o note=null
4) 중복 방지(중요)
• WorkOrderChecklistItem은 “한 번만” 생성되어야 한다.
• 구현 방법(택1):
o (권장) work_order_id 단위로 “이미 체크리스트가 있으면 생성 스킵”
o 또는 (work_order_id, sort_order) / (work_order_id, label) 유니크로 중복 insert 방지
5) 항목 수정 정책(Phase 1)
• 생성된 체크리스트 항목의 label/required/sort_order는 운영 중 변경하지 않는다(읽기 전용 취급).
• 기사가 수정할 수 있는 필드는 value/note만.
6) 상태/권한 연동(핵심)
• 체크리스트 입력(PATCH)은 다음 조건에서만 허용:
o 역할: TECHNICIAN
o 스코프: SELF(본인 배정)
o 상태: != COMPLETED && != CANCELLED
7) FAIL 메모 규칙
• value=FAIL일 때 note는 필수(빈 값 불가).
• value!=FAIL이면 note는 선택(기존 note가 있으면 유지 가능).
8) 완료 제출 검증
• POST /workorders/{id}/complete 서버 검증에서:
o required 항목 전체가 value != null
o FAIL 항목은 note 존재
o 서명 존재(WorkOrderSignature)
o (옵션) 사진 최소 조건(정책 토글)
9) 체크리스트 “버전” 처리(Phase 1 단순화)
• 체크리스트 항목이 바뀌어도 기존 WorkOrder에는 영향을 주지 않는다.
• 필요해지는 시점(Phase 2)에 ChecklistTemplate/버전 관리로 확장한다.
10) 실무 팁(문서에 한 줄)
• “필수 항목 수를 줄이거나 문구를 바꿀 경우, 신규 생성되는 WorkOrder부터 적용한다.”


체크리스트 고정 항목 세트 (Phase 1)
A) INSTALL (필수 12개)
1. (required=true) 설치 위치가 협의된 위치와 일치한다
2. (required=true) 전원 사용 가능(콘센트/차단기/연결 상태)
3. (required=true) 설치 공간 안전/간섭(통행, 고정 가능 공간) 확보
4. (required=true) 제품 모델이 지시 내용과 일치한다
5. (required=true) 제품 시리얼이 확인되었다(실물 기준)
6. (required=true) 외관 이상(파손/심각한 오염)이 없다
7. (required=true) 고정(브라켓/거치/고정장치)이 견고하게 완료되었다
8. (required=true) 수평/정렬이 허용 범위 내이다(육안 기준 OK)
9. (required=true) 전원 인가 후 정상 동작한다(부팅/동작 확인)
10. (required=true) 이상 알림/에러 표시가 없다(또는 처리 완료)
11. (required=true) 고객에게 기본 사용법/주의사항을 안내했다
12. (required=true) 고객 서명 완료(서명자 이름 포함) (UI에서는 서명 섹션 완료와 연동해 자동 ✅ 처리 권장)

B) SERVICE / AS (필수 10개)
1. (required=true) 접수 증상/요청을 고객과 재확인했다
2. (required=true) 대상 제품 모델을 확인했다
3. (required=true) 대상 제품 시리얼을 확인했다
4. (required=true) 전원/연결 상태를 점검했다
5. (required=true) 증상이 재현되었거나 원인이 특정되었다(미재현이면 그 사실 기록)
6. (required=true) 필요한 조치(수리/설정/청소/교체 등)를 수행했다
7. (required=true) 조치 후 동일 증상이 개선/해소되었음을 확인했다
8. (required=true) 고객에게 처리 내용/주의사항을 설명했다
9. (required=true) 추가 방문/추가 조치 필요 여부를 안내했다(필요 시)
10. (required=true) 고객 서명 완료(서명자 이름 포함) (UI에서는 서명 섹션 완료와 연동해 자동 ✅ 처리 권장)

구현 메모(개발자용, 짧게)
• WorkOrder 생성 시 type에 맞는 리스트를 WorkOrderChecklistItem으로 insert
• value/note는 null로 시작
• 완료 제출 시 “서명 항목(마지막 항목)”은 체크리스트 값으로 검사하지 말고, WorkOrderSignature 존재로 검증하는 게 더 안전함
o (그래도 항목은 남겨서 사용자에게 ‘서명은 필수’임을 반복 노출)

(본문 끝)
===

WorkOrder 데이터 항목 정의(Field Dictionary) v0.1 — Phase 1
1) 기본 식별/분류
필드
타입
필수
입력 주체
예시
제약/비고
id
string/uuid
✅
시스템
WO-1042
표시용 ID 포맷은 선택
type
enum
✅
Admin
INSTALL / SERVICE
DRAFT에서만 변경 권장
status
enum
✅
시스템
TEAM_ASSIGNED
상태 전이 규칙 준수
priority
enum
⭕
Admin
NORMAL
LOW/NORMAL/HIGH

2) 고객/현장 참조
필드
타입
필수
입력 주체
예시
제약/비고
customer_id
fk
✅
Admin
CUST-22
고객 마스터
site_id
fk
✅
Admin
SITE-108
현장 마스터(주소 포함)
(표시) customer_name
derived
-
시스템
OO유통(주)
응답에 포함 권장
(표시) site_address
derived
-
시스템
서울시 …
응답에 포함 권장

3) 작업 내용
필드
타입
필수
입력 주체
예시
제약/비고
summary
text(short)
✅
Admin
공간살균기 설치
50자 내 권장
description
text
⭕
Admin
출입문 오른쪽…
특이사항/주의사항
scheduled_start_at
datetime
⭕
Admin
2026-02-12 14:00
지연 계산 기준
product_serial_text
text
⭕
Admin/기사(정책)
SN1234…
Phase 1은 텍스트로만

4) 배정(Assignment)
필드
타입
필수
입력 주체
예시
제약/비고
assigned_team_id
fk
⭕(흐름상 필요)
Admin
TEAM-3
팀 미배정이면 null
assigned_technician_id
fk
⭕(흐름상 필요)
Team Manager
U-55
기사 미배정이면 null
권장 정책:
• Admin은 팀까지만 배정(기사 배정은 TM 담당)
• IN_PROGRESS 이후 팀/기사 변경은 제한 + 사유(정책)

5) 완료/취소 메타
필드
타입
필수
입력 주체
예시
제약/비고
started_at
datetime
⭕
시스템
2026-02-12 14:05
start 시점 기록
completed_at
datetime
⭕
시스템
2026-02-12 14:40
complete 시점
cancelled_at
datetime
⭕
시스템
2026-02-12 13:10
cancel 시점
cancel_reason
text
⭕
Admin
고객 일정 변경
취소 시 필수

6) 고객 발송 설정(옵션, Phase 1 최소)
필드
타입
필수
입력 주체
예시
제약/비고
notify_sms_enabled
bool
⭕
Admin
true
Phase 1 기본 OFF 권장
notify_email_enabled
bool
⭕
Admin
true
Phase 1 기본 OFF 권장
customer_notify_phone
string
⭕
Admin
01012345678
번호 포맷 검증 필요
customer_notify_email
string
⭕
Admin
a@b.com
이메일 검증 필요
Phase 1에서는 발송 “자동/재발송”을 최소로 하고, 실패 시 Admin이 처리하는 흐름 권장.

7) 시스템 필드
필드
타입
필수
입력 주체
예시
제약/비고
created_by_user_id
fk
✅
시스템
U-1
생성자
created_at
datetime
✅
시스템
-

updated_at
datetime
✅
시스템
-


서버 검증 규칙(WorkOrder 단)
1. complete 시점에 체크리스트 필수 입력 + FAIL 메모 + 서명 존재를 검증한다.
2. cancel은 사유 필수, 권장 정책상 COMPLETED는 취소 불가.
3. 스코프/권한(ORG/TEAM/SELF)은 모든 조회/수정 요청에서 우선 검증한다.

(본문 끝)
===
