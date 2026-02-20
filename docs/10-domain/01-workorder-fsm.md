# WorkOrder FSM

## TL;DR
- WorkOrder 상태(DRAFT→…→COMPLETED)와 완료 후 서버 자동처리(PDF/발송/Audit)를 v0.1 기준으로 고정.

## 본문
1) WorkOrder 상태 다이어그램
상태 정의
• DRAFT: 작업 생성만 됨(임시 저장 포함)
• TEAM_ASSIGNED: 본사가 팀을 배정함
• TECH_ASSIGNED: 팀 관리자가 기사를 배정함
• IN_PROGRESS: 기사가 작업 시작(선택 상태지만 추천)
• COMPLETED: 체크리스트/서명 완료 제출됨 (PDF 생성/발송 트리거)
• CANCELLED: 취소(사유 기록)
전이 규칙
DRAFT
  └─(Admin: 팀 배정)→ TEAM_ASSIGNED
TEAM_ASSIGNED
  └─(TeamManager: 기사 배정)→ TECH_ASSIGNED
TECH_ASSIGNED
  └─(Technician: 작업 시작)→ IN_PROGRESS
IN_PROGRESS
  └─(Technician: 체크리스트 완료 + 서명 필수 + 제출)→ COMPLETED

[취소]
DRAFT / TEAM_ASSIGNED / TECH_ASSIGNED / IN_PROGRESS
  └─(Admin 또는 권한있는 TeamManager)→ CANCELLED
완료(Completed) 시 자동 처리(서버 규칙)
• 서명 존재 여부 검증(필수)
• (선택) 필수 체크리스트 항목 검증
• PDF 생성 → 파일 저장
• 고객 문자/이메일 발송
• AuditLog 기록(성공/실패 포함)

좋아 쭌. 아래는 **Phase 1 – 화면 목록 + 기능 매핑표(페이지 스펙)**이야.
개발자/외주에게 그대로 주면 “무엇을 어떤 화면에서 구현해야 하는지” 바로 이해할 수 있는 형태로 정리했어.
(ERD 초안 기준 테이블명을 괄호로 같이 매핑했음)

2) 화면 목록 + 기능 매핑표
공통(모든 역할)
P-00. 로그인
• 목적: 사용자 인증
• 기능
o 로그인(이메일/비밀번호)
o 비활성 계정 차단
• 데이터
o Read: User
• 검증/정책
o Role 기반으로 메뉴/접근 제어

P-01. 내 정보(프로필)
• 목적: 사용자 기본정보 확인
• 기능
o 이름/전화/이메일 조회(초기엔 수정 제한 가능)
• 데이터
o Read: User

본사 관리자(Admin) 화면
A-01. 관리자 대시보드
• 목적: 오늘 작업/지연/미배정 현황을 한눈에
• 기능
o KPI 카드: 오늘 작업 수, 진행중, 지연, 완료
o 팀별 현황(건수)
o 클릭 시 작업 목록 필터로 이동
• 데이터
o Read: WorkOrder, Team

## 연관 문서
- [화면 지도](../20-product/01-screen-map.md)
- [M-02 작업 상세(기사)](../30-implementation/01-m02-spec.md)

## 변경 이력
- v0.1: 통합문서 기반 초안 정리
