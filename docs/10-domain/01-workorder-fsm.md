# WorkOrder FSM

## TL;DR
- WorkOrder 상태(DRAFT→…→COMPLETED)와 완료 후 서버 자동처리(PDF/발송/Audit)를 v0.1 기준으로 정의합니다.

## 본문

### 1) WorkOrder 상태 정의
| 상태명 | 설명 | 비고 |
| :--- | :--- | :--- |
| **DRAFT** | 작업 생성 직후 상태 (임시 저장 포함) | - |
| **TEAM_ASSIGNED** | 본사(Admin)가 특정 팀을 배정 완료 | - |
| **TECH_ASSIGNED** | 팀 관리자가 특정 기사를 배정 완료 | - |
| **IN_PROGRESS** | 기사가 모바일에서 '작업 시작'을 누름 | 권장 필수 단계 |
| **COMPLETED** | 체크리스트/서명 제출 완료 | PDF/발송 트리거 |
| **CANCELLED** | 작업 취소 | 사유 기록 필수 |

---

### 2) 상태 전이 규칙 (Flow)
1. **DRAFT**
   - └─ `Admin: 팀 배정` ➔ **TEAM_ASSIGNED**
2. **TEAM_ASSIGNED**
   - └─ `Team Manager: 기사 배정` ➔ **TECH_ASSIGNED**
3. **TECH_ASSIGNED**
   - └─ `Technician: 작업 시작` ➔ **IN_PROGRESS**
4. **IN_PROGRESS**
   - └─ `Technician: 체크리스트 완료 + 서명 ➔ 제출` ➔ **COMPLETED**

**[취소(CANCELLED) 규칙]**
- **DRAFT / TEAM_ASSIGNED / TECH_ASSIGNED / IN_PROGRESS** 상태에서 Admin 또는 권한 있는 Team Manager가 실행 가능.

---

### 3) 완료(COMPLETED) 시 서버 자동 처리
시스템에서 완료 제출 시 아래 항목을 순차적으로 처리합니다.
- [ ] **데이터 검증:** 서명 파일 존재 여부 및 필수 체크리스트 항목 확인
- [ ] **PDF 생성:** 최종 작업 보고서 PDF 생성 및 스토리지 저장
- [ ] **고객 발송:** 설정된 정보로 문자/이메일 자동 발송
- [ ] **감사 로그:** 모든 처리 과정의 성공/실패 여부를 AuditLog에 기록

---

### 4) 화면 목록 및 기능 매핑
| 역할 | 화면 ID | 화면명 | 주요 기능 | 연관 데이터 |
| :--- | :---: | :--- | :--- | :--- |
| **공통** | P-00 | 로그인 | 이메일/PW 인증, Role 기반 제어 | User |
| **공통** | P-01 | 내 정보 | 프로필 조회, 비밀번호 관리 | User |
| **Admin** | A-01 | 관리자 대시보드 | 작업/지연/미배정 현황 KPI | WorkOrder, Team |

---

## 연관 문서
- [화면 지도](../20-product/01-screen-map.md)
- [M-02 작업 상세(기사)](../30-implementation/01-m02-spec.md)

## 변경 이력
- v0.1: 통합문서 기반 초안 정리
