# M-02 작업 상세(기사) 구현 스펙

## TL;DR
- **목적:** 현장 기사 전용 모바일 웹 화면의 기술적 구현 표준 정의
- **핵심 정책:** 실시간 자동 저장(Autosave), 강력한 데이터 검증(Validation), 비동기 사진 업로드

---

## 🏗️ 1. 기술적 구조 (Technical Structure)

### 🧩 화면 구성 및 상태 관리
- **Sticky Header:** 작업 타입 및 실시간 상태 배지 표시
- **Sticky Footer:** 완료 제출 버튼 (검증 로직 포함)
- **Autosave:** 사용자가 체크리스트 항목을 선택하거나 텍스트를 입력하면 디바운스(Debounce) 또는 즉시 API 호출을 통해 상태를 동기화합니다.

---

## 📋 2. 상세 구현 스펙

### ✅ 체크리스트 (Checklist)
- **데이터 모델:** `WorkOrderChecklistItem` (ID, Value, Note, Order)
- **Interaction:**
  - `FAIL` 선택 시 `Note` 필드 가시성 ➔ `True` 전환
  - 섹션별 진행률(`7/12`) 실시간 계산 및 UI 반영
- **Validation:** 제출 시 모든 `is_required=true` 항목의 `value != null` 확인

### ✍️ 고객 서명 (Digital Signature)
- **기술 스택:** HTML5 Canvas 기반 라이브러리 (e.g., `signature_pad`)
- **저장 프로세스:** [서명 저장] 버튼 클릭 시 Base64 또는 Blob 데이터를 `FileObject`로 업로드 후 `signature_file_id`를 `WorkOrder`와 매핑
- **Constraint:** 서명 파일 식별자가 없는 경우 완료 제출 차단

### 📸 사진 업로드 (Attachments)
- **Category:** `BEFORE_PHOTO`, `AFTER_PHOTO`
- **UI:** 썸네일 그리드 + 업로드 상태 인디케이터 + 개별 삭제 기능
- **Retry:** 업로드 실패 시 해당 항목에 [재시도] 버튼 노출

---

## 🏁 3. 완료 제출 로직 (Submit Logic)

### 🛡️ 클라이언트 사이드 검증 (Validation)
1. **필수값 체크:** 체크리스트 내 모든 필수 항목 응답 여부
2. **조건부 필수:** `FAIL` 항목의 메모(Note) 입력 여부
3. **서명 확인:** 서명 데이터의 서버 저장 완료 여부
4. **포맷 확인:** 고객 발송용 연락처 및 이메일 형식 유효성

### 🚀 서버 사이드 프로세스 (Post-submission)
1. `WorkOrder.status` ➔ `COMPLETED` 업데이트
2. `completed_at` 타임스탬프 기록
3. PDF 생성 워커(Worker) 트리거 및 고객 발송 큐(Queue) 등록

---

## 📂 연관 문서 및 리스트
- **API Spec:** [03-api.md](03-api.md)
- **컴포넌트 리스크:** [02-m02-components-risk.md](02-m02-components-risk.md)
- **기사 화면 정의:** [../20-product/04-technician.md](../20-product/04-technician.md)

## 변경 이력
- **v0.1:** 기획-구현 통합 기반 기술 스펙 초안 (2025-02-20)
