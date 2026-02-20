# 권한 매트릭스(RBAC)

## TL;DR
- 화면/행위 단위 권한과 API 레벨 권한을 분리해 정의합니다.

## 본문
목표는 “운영 혼선 방지 + 책임소재 명확화”라서, Admin은 컨트롤/감사, Team Manager는 배정/관리, Technician은 수행/증빙으로 딱 나눴어.
표기:
• ✅ 허용
• ⚠️ 제한/조건부(사유 필요, 상태 제한 등)
• ❌ 불가

Phase 1 권한 매트릭스 (RBAC)
1) 인증/계정
기능
Admin
Team Manager
Technician
로그인/로그아웃
✅
✅
✅
내 정보 조회
✅
✅
✅
내 정보 수정(이메일/전화)
⚠️ (정책)
⚠️ (정책)
⚠️ (정책)
비밀번호 변경/재설정
✅
✅
✅
사용자 생성/비활성화
✅
❌
❌
역할 변경(ADMIN/TEAM/TECH)
✅
❌
❌
팀 생성/수정/비활성화
✅
❌
❌
사용자-팀 연결 변경
✅
❌
❌
Phase 1에서는 “사용자 프로필 수정”은 혼선이 많아서 초기엔 제한 추천(필요 시 Admin만 수정).

2) 고객/현장(Customer/Site)
기능
Admin
Team Manager
Technician
고객 목록/상세 조회
✅
⚠️ (팀 작업 관련만)
⚠️ (본인 작업 관련만)
고객 생성/수정
✅
❌ (또는 ⚠️)
❌
현장 목록/상세 조회
✅
⚠️ (팀 작업 관련만)
⚠️ (본인 작업 관련만)
현장 생성/수정
✅
❌ (또는 ⚠️)
❌
추천 운영:
• 고객/현장은 “마스터 데이터”로 보고 Admin만 수정이 안정적.
• 현장 연락처가 자주 바뀌면 Team Manager에게 수정 권한을 ⚠️로 열어도 됨(변경 로그 필수).

3) WorkOrder 생성/수정/조회 (핵심)
기능
Admin
Team Manager
Technician
작업 목록 조회
✅ (전체)
✅ (팀 범위)
✅ (본인 배정)
작업 상세 조회
✅ (전체)
✅ (팀 범위)
✅ (본인 배정)
작업 생성(설치/AS)
✅
⚠️ (정책: 허용 시 팀 범위만)
❌
작업 내용 수정(summary/description)
⚠️ (DRAFT~TEAM_ASSIGNED)
❌
❌
작업 일정 수정
⚠️ (DRAFT~TECH_ASSIGNED)
⚠️ (TEAM_ASSIGNED~TECH_ASSIGNED, 정책)
❌
우선순위 변경
⚠️ (DRAFT~TECH_ASSIGNED)
❌
❌
작업 타입 변경(INSTALL/AS)
⚠️ (DRAFT만)
❌
❌
작업 취소(CANCELLED)
✅ (사유 필수)
⚠️ (정책: TEAM_ASSIGNED까지만 + 사유)
❌
강력 추천:
• 내용/일정/타입 변경은 “완료 후 분쟁”을 부르므로 상태 제한을 문서에 명시.

4) 배정(본사→팀→기사) (Phase 1 핵심 체인)
기능
Admin
Team Manager
Technician
팀 배정(assigned_team_id)
✅
❌
❌
팀 변경
⚠️ (IN_PROGRESS 이후 제한 + 사유)
❌
❌
기사 배정(assigned_technician_id)
❌(기본) / ⚠️(예외)
✅
❌
기사 재배정
❌(기본) / ⚠️(예외)
⚠️ (IN_PROGRESS 제한 + 사유)
❌
Admin이 기사까지 배정하면 운영 체계가 꼬이기 쉬워서 기본은 금지가 안정적. “긴급 모드”가 필요하면 예외 권한(⚠️)로만.

5) 작업 수행/증빙(기사 영역)
기능
Admin
Team Manager
Technician
작업 시작(IN_PROGRESS)
❌
❌
⚠️ (TECH_ASSIGNED에서만)
체크리스트 입력/수정
❌
❌
⚠️ (완료 전까지만)
FAIL 메모 입력
❌
❌
✅ (FAIL 시 필수)
사진 업로드/삭제
❌
❌
⚠️ (완료 전까지만)
서명 입력/저장
❌
❌
✅ (필수)
완료 제출(COMPLETED)
❌
❌
✅ (검증 통과 시)
보조 권한(읽기):
• Admin/Team Manager는 체크리스트/사진/서명 조회만 ✅

6) 문서(PDF) / 발송(SMS/Email)
기능
Admin
Team Manager
Technician
PDF 생성(트리거)
시스템(완료 시 자동)
시스템
시스템
PDF 다운로드/열람
✅
✅ (팀 범위)
⚠️ (본인 작업, 읽기)
고객 발송 설정(번호/이메일/옵션)
✅
⚠️ (정책)
⚠️ (정책)
고객 발송 실행(완료 후 자동)
시스템
시스템
시스템
재발송(문자/이메일)
✅
⚠️ (정책: 팀 범위만)
❌
발송 실패 원인 확인
✅
⚠️ (팀 범위)
⚠️ (간단 상태만)
추천:
• 발송 옵션(번호/이메일)은 “현장 실수”를 막기 위해 초기엔 Admin이 세팅하고, 현장에선 읽기만 하는 것도 안정적.

7) 이력/감사(AuditLog)
기능
Admin
Team Manager
Technician
AuditLog 조회(전체)
✅
⚠️ (팀 범위)
❌
작업 단위 히스토리(요약)
✅
✅ (팀 범위)
⚠️ (본인 작업의 간단 로그)

8) 데이터 스코프 규칙(가장 중요, 문서에 박아두기)
• Admin: Organization 전체
• Team Manager: assigned_team_id = 내 team_id 인 WorkOrder 및 관련 Customer/Site/Attachments
• Technician: assigned_technician_id = 내 user_id 인 WorkOrder 및 관련 데이터만

Phase 1 “권장 정책값” (결정안으로 써도 됨)
• 고객/현장 마스터 수정: Admin만
• 기사 배정: Team Manager만
• 팀 변경: Admin만, IN_PROGRESS 이후 제한 + 사유 필수
• 완료 이후 수정: 핵심 증빙(체크리스트/서명/사진) 수정 금지
• 재발송: Admin 기본, Team Manager는 필요 시 팀 범위로만
목표는 “운영 혼선 방지 + 책임소재 명확화”라서, Admin은 컨트롤/감사, Team Manager는 배정/관리, Technician은 수행/증빙으로 딱 나눴어.
표기:
• ✅ 허용
• ⚠️ 제한/조건부(사유 필요, 상태 제한 등)
• ❌ 불가

Phase 1 권한 매트릭스 (RBAC)
1) 인증/계정
기능
Admin
Team Manager
Technician
로그인/로그아웃
✅
✅
✅
내 정보 조회
✅
✅
✅
내 정보 수정(이메일/전화)
⚠️ (정책)
⚠️ (정책)
⚠️ (정책)
비밀번호 변경/재설정
✅
✅
✅
사용자 생성/비활성화
✅
❌
❌
역할 변경(ADMIN/TEAM/TECH)
✅
❌
❌
팀 생성/수정/비활성화
✅
❌
❌
사용자-팀 연결 변경
✅
❌
❌
Phase 1에서는 “사용자 프로필 수정”은 혼선이 많아서 초기엔 제한 추천(필요 시 Admin만 수정).

2) 고객/현장(Customer/Site)
기능
Admin
Team Manager
Technician
고객 목록/상세 조회
✅
⚠️ (팀 작업 관련만)
⚠️ (본인 작업 관련만)
고객 생성/수정
✅
❌ (또는 ⚠️)
❌
현장 목록/상세 조회
✅
⚠️ (팀 작업 관련만)
⚠️ (본인 작업 관련만)
현장 생성/수정
✅
❌ (또는 ⚠️)
❌
추천 운영:
• 고객/현장은 “마스터 데이터”로 보고 Admin만 수정이 안정적.
• 현장 연락처가 자주 바뀌면 Team Manager에게 수정 권한을 ⚠️로 열어도 됨(변경 로그 필수).

3) WorkOrder 생성/수정/조회 (핵심)
기능
Admin
Team Manager
Technician
작업 목록 조회
✅ (전체)
✅ (팀 범위)
✅ (본인 배정)
작업 상세 조회
✅ (전체)
✅ (팀 범위)
✅ (본인 배정)
작업 생성(설치/AS)
✅
⚠️ (정책: 허용 시 팀 범위만)
❌
작업 내용 수정(summary/description)
⚠️ (DRAFT~TEAM_ASSIGNED)
❌
❌
작업 일정 수정
⚠️ (DRAFT~TECH_ASSIGNED)
⚠️ (TEAM_ASSIGNED~TECH_ASSIGNED, 정책)
❌
우선순위 변경
⚠️ (DRAFT~TECH_ASSIGNED)
❌
❌
작업 타입 변경(INSTALL/AS)
⚠️ (DRAFT만)
❌
❌
작업 취소(CANCELLED)
✅ (사유 필수)
⚠️ (정책: TEAM_ASSIGNED까지만 + 사유)
❌
강력 추천:
• 내용/일정/타입 변경은 “완료 후 분쟁”을 부르므로 상태 제한을 문서에 명시.

4) 배정(본사→팀→기사) (Phase 1 핵심 체인)
기능
Admin
Team Manager
Technician
팀 배정(assigned_team_id)
✅
❌
❌
팀 변경
⚠️ (IN_PROGRESS 이후 제한 + 사유)
❌
❌
기사 배정(assigned_technician_id)
❌(기본) / ⚠️(예외)
✅
❌
기사 재배정
❌(기본) / ⚠️(예외)
⚠️ (IN_PROGRESS 제한 + 사유)
❌
Admin이 기사까지 배정하면 운영 체계가 꼬이기 쉬워서 기본은 금지가 안정적. “긴급 모드”가 필요하면 예외 권한(⚠️)로만.

5) 작업 수행/증빙(기사 영역)
기능
Admin
Team Manager
Technician
작업 시작(IN_PROGRESS)
❌
❌
⚠️ (TECH_ASSIGNED에서만)
체크리스트 입력/수정
❌
❌
⚠️ (완료 전까지만)
FAIL 메모 입력
❌
❌
✅ (FAIL 시 필수)
사진 업로드/삭제
❌
❌
⚠️ (완료 전까지만)
서명 입력/저장
❌
❌
✅ (필수)
완료 제출(COMPLETED)
❌
❌
✅ (검증 통과 시)
보조 권한(읽기):
• Admin/Team Manager는 체크리스트/사진/서명 조회만 ✅

6) 문서(PDF) / 발송(SMS/Email)
기능
Admin
Team Manager
Technician
PDF 생성(트리거)
시스템(완료 시 자동)
시스템
시스템
PDF 다운로드/열람
✅
✅ (팀 범위)
⚠️ (본인 작업, 읽기)
고객 발송 설정(번호/이메일/옵션)
✅
⚠️ (정책)
⚠️ (정책)
고객 발송 실행(완료 후 자동)
시스템
시스템
시스템
재발송(문자/이메일)
✅
⚠️ (정책: 팀 범위만)
❌
발송 실패 원인 확인
✅
⚠️ (팀 범위)
⚠️ (간단 상태만)
추천:
• 발송 옵션(번호/이메일)은 “현장 실수”를 막기 위해 초기엔 Admin이 세팅하고, 현장에선 읽기만 하는 것도 안정적.

7) 이력/감사(AuditLog)
기능
Admin
Team Manager
Technician
AuditLog 조회(전체)
✅
⚠️ (팀 범위)
❌
작업 단위 히스토리(요약)
✅
✅ (팀 범위)
⚠️ (본인 작업의 간단 로그)

8) 데이터 스코프 규칙(가장 중요, 문서에 박아두기)
• Admin: Organization 전체
• Team Manager: assigned_team_id = 내 team_id 인 WorkOrder 및 관련 Customer/Site/Attachments
• Technician: assigned_technician_id = 내 user_id 인 WorkOrder 및 관련 데이터만

Phase 1 “권장 정책값” (결정안으로 써도 됨)
• 고객/현장 마스터 수정: Admin만
• 기사 배정: Team Manager만
• 팀 변경: Admin만, IN_PROGRESS 이후 제한 + 사유 필수
• 완료 이후 수정: 핵심 증빙(체크리스트/서명/사진) 수정 금지
• 재발송: Admin 기본, Team Manager는 필요 시 팀 범위로만

(본문 끝)
===

(개발할 때 백엔드에서 그대로 권한 가드(authorization middleware) 규칙으로 쓰기 좋게)
표기:
• Roles: A(Admin) / TM(Team Manager) / T(Technician)
• Scope:
o ORG: 전체
o TEAM: assigned_team_id == TM.team_id
o SELF: assigned_technician_id == T.id
• Status guard: [] 안에 허용 상태

Phase 1 API 권한/상태 가드 규칙표
0) 공통 가드(모든 엔드포인트)
• 인증 필수(로그인)
• 모든 WorkOrder 관련 요청은 먼저 스코프 검증:
o A: ORG
o TM: TEAM
o T: SELF

1) Auth / User / Team
Endpoint
허용 Role
Scope
상태
비고
POST /auth/login
A/TM/T
-
-
로그인
POST /auth/logout
A/TM/T
-
-
로그아웃
GET /me
A/TM/T
SELF
-
내 정보
PATCH /me
⚠️(정책)
SELF
-
Phase 1은 제한 권장
GET /admin/teams?active=true
A
ORG
-
팀 목록
POST /admin/teams
A
ORG
-
팀 생성
PATCH /admin/teams/{id}
A
ORG
-
팀 수정/비활성
POST /admin/users
A
ORG
-
사용자 생성
PATCH /admin/users/{id}
A
ORG
-
사용자/역할/팀 변경
GET /teams/{teamId}/technicians
TM/A
TEAM/ORG
-
기사 드롭다운

2) Customer / Site (마스터 데이터)
Endpoint
허용 Role
Scope
상태
비고
GET /customers
A
ORG
-
전체 고객
GET /customers/{id}
A
ORG
-
고객 상세
POST /customers
A
ORG
-
고객 생성
PATCH /customers/{id}
A
ORG
-
고객 수정
GET /sites/{id}
A
ORG
-
현장 상세
POST /sites
A
ORG
-
현장 생성
PATCH /sites/{id}
A
ORG
-
현장 수정
TM/T는 “작업 상세 응답에 포함된 고객/현장 읽기”만 허용하는 편이 안전(별도 endpoint 차단).

3) WorkOrder 목록/상세
Endpoint
허용 Role
Scope
상태
비고
GET /admin/workorders?preset=...
A
ORG
-
A-02 목록
GET /teams/{teamId}/workorders?preset=...
TM
TEAM
-
T-02 목록
GET /tech/workorders?date=...
T
SELF
-
M-01 목록
GET /workorders/{id}
A/TM/T
ORG/TEAM/SELF
-
상세(각자 스코프)

4) WorkOrder 생성/수정/취소
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders
A
ORG
-
생성(기본 Admin)
PATCH /workorders/{id}
A
ORG
[DRAFT, TEAM_ASSIGNED, TECH_ASSIGNED]
내용/일정/우선순위
POST /workorders/{id}/cancel
A
ORG
!= COMPLETED
취소 사유 필수
PATCH /workorders/{id}에서 “무엇을 수정 가능”은 필드별로 추가 가드 권장(예: type은 DRAFT만).

5) 본사→팀 배정(핵심)
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders/{id}/assign-team
A
ORG
[DRAFT, TEAM_ASSIGNED, TECH_ASSIGNED]
팀 배정/변경
POST /workorders/{id}/change-team (선택)
A
ORG
NOT IN [IN_PROGRESS, COMPLETED]
변경 사유 필수 권장
단일 endpoint로 합치고, 팀 변경 시 reason 요구하는 것도 OK.

6) 팀→기사 배정(핵심)
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders/{id}/assign-technician
TM
TEAM
[TEAM_ASSIGNED, TECH_ASSIGNED]
기사 배정/재배정
POST /workorders/{id}/reassign-technician (선택)
TM
TEAM
!= COMPLETED + 정책
재배정 사유 권장
(예외) POST /workorders/{id}/assign-technician
A(⚠️)
ORG
정책
긴급 모드만

7) 기사 수행(체크리스트/서명/사진/완료)
7.1 작업 시작
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders/{id}/start
T
SELF
[TECH_ASSIGNED]
IN_PROGRESS 전환
7.2 체크리스트
Endpoint
허용 Role
Scope
상태 가드
비고
GET /workorders/{id}/checklist-items
A/TM/T
ORG/TEAM/SELF
-
조회
PATCH /workorders/{id}/checklist-items/{itemId}
T
SELF
NOT IN [COMPLETED, CANCELLED]
autosave
PATCH /workorders/{id}/checklist-items (선택)
T
SELF
NOT IN [COMPLETED, CANCELLED]
배치 저장
7.3 서명
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders/{id}/signature/upload-url
T
SELF
NOT IN [COMPLETED, CANCELLED]
업로드 URL
POST /workorders/{id}/signature
T
SELF
NOT IN [COMPLETED, CANCELLED]
서명 등록
DELETE /workorders/{id}/signature (선택)
T
SELF
NOT IN [COMPLETED, CANCELLED]
재입력
7.4 사진
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders/{id}/photos/upload-url
T
SELF
NOT IN [COMPLETED, CANCELLED]
업로드 URL
POST /workorders/{id}/attachments
T
SELF
NOT IN [COMPLETED, CANCELLED]
첨부 등록
DELETE /workorders/{id}/attachments/{attachmentId}
T
SELF
NOT IN [COMPLETED, CANCELLED]
삭제(정책)
GET /workorders/{id}/attachments
A/TM/T
ORG/TEAM/SELF
-
조회
7.5 완료 제출
Endpoint
허용 Role
Scope
상태 가드
비고
POST /workorders/{id}/complete
T
SELF
[TECH_ASSIGNED, IN_PROGRESS]
서버 검증 필수

8) 문서(PDF)/발송 상태/재발송
Endpoint
허용 Role
Scope
상태 가드
비고
GET /workorders/{id}/delivery-status
A/TM/T
ORG/TEAM/SELF
-
상태 조회
GET /workorders/{id}/pdf
A/TM/T
ORG/TEAM/SELF
status=COMPLETED
다운로드
POST /workorders/{id}/resend
A
ORG
status=COMPLETED
문자/이메일 재발송
(선택) POST /workorders/{id}/pdf/regenerate
A
ORG
status=COMPLETED
PDF 재생성

9) AuditLog
Endpoint
허용 Role
Scope
상태
비고
GET /workorders/{id}/auditlogs
A/TM
ORG/TEAM
-
TM은 팀 범위만
(선택) GET /admin/auditlogs?filter=...
A
ORG
-
전체 감사

“결정해야 하는 정책 토글” 5개(Phase 1 옵션)
문서 마지막에 체크박스로 붙여두면 좋아.
1. Admin이 기사 배정까지 가능한가? (기본: ❌)
2. Team Manager가 취소 가능한가? (기본: ❌ 또는 TEAM_ASSIGNED까지만)
3. 일정 변경 권한을 TM에게 줄까? (기본: 제한적 ⚠️)
4. 사진을 완료 조건에 포함할까? (초기: ❌, 추후 토글)
5. 완료 후 수정(체크리스트/서명/사진) 허용할까? (기본: ❌)

(본문 끝)
===