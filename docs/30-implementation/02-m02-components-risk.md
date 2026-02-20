# M-02 컴포넌트/난이도/리스크

## TL;DR
- M-02 구현 요소를 컴포넌트 단위로 쪼개고 리스크(서명/사진/자동저장)를 선제 관리.

## 본문
## M-02 구현 컴포넌트 목록, 난이도,리스크

M-02 구현 컴포넌트 목록 + 난이도/리스크
0) 화면 구성 트리(컴포넌트 계층)
• WorkOrderDetailPage (M-02)
o StickyHeader
o SiteCard
o WorkSummaryCard (선택)
o ChecklistSection
• ChecklistItemRow × N
• FailMemoInput (조건부)
o SignatureSection
• SignerNameInput
• SignerRoleSelect (선택)
• SignaturePad
• SignaturePreview (저장 후)
o ExpandableSection × 3
• PhotoUploadSection
• OptionalItemsSection (필요 시)
• CustomerNotifySection
o StickyActionBar
o Toast/AlertSystem
o ConfirmModal (선택)

1) UI 컴포넌트 상세
C1. StickyHeader
• 역할: 뒤로가기 + 상태 표시 + 즉시 전화
• 난이도: 낮음
• 주의: 상태 배지/타입 배지는 서버 값 기반

C2. SiteCard
• 역할: 현장 핵심정보 + 전화/지도
• 난이도: 낮음
• 주의: 주소 탭하면 지도앱 이동(링크 정책 1개로 통일)

C3. WorkSummaryCard (선택)
• 역할: 일정/특이사항
• 난이도: 낮음
• 주의: “자세히 보기/접기”만 제공, 긴 텍스트는 줄임표 처리

C4. ChecklistSection (핵심)
• 역할: 필수 10~12개를 가장 빠르게 입력
• 난이도: 중간
• 리스크 포인트
o 입력 UX(토글 버튼) 품질
o 자동 저장(autosave) 타이밍
o 미완료/에러 표시
• 필수 규칙
o 각 항목: PASS/FAIL/NA 중 1개 선택
o FAIL이면 메모 입력창 자동 펼침 + 메모 필수
o 진행률 “완료 X/N” 실시간 계산
• 구현 팁
o 항목 클릭 즉시 optimistic update(화면 반영) + 백그라운드 저장
o 저장 실패 시 해당 항목에 “저장 실패·재시도” 표시

C5. ChecklistItemRow
• 난이도: 중간
• 상태
o value = null (미입력)
o value = PASS|FAIL|NA
o save_status = saved|saving|error
• UI
o 버튼 3개는 “세그먼트 컨트롤” 형태 추천

C6. FailMemoInput
• 난이도: 낮음~중간
• 필수 규칙
o FAIL이면 note가 비어있으면 완료 불가
o note 최소 길이(정책으로 0→10자) 조절 가능하게

C7. SignatureSection (핵심)
• 역할: 서명 수집/저장/미리보기
• 난이도: 중간~높음 (가장 까다로움)
• 리스크 포인트
o 모바일 브라우저에서 서명 캔버스 입력 품질
o 터치 이벤트(iOS/Android) 호환
o 저장된 이미지 품질(너무 얇거나 깨짐)
• 필수 규칙
o 서명 저장 전 “완료 제출” 불가
o 서명자 이름 필수
o 저장 후 미리보기 + “다시 받기” 제공
• 구현 팁
o 캔버스 해상도를 실제 CSS 크기보다 크게(레티나 대응)
o 저장 포맷은 PNG 권장(투명 배경 가능)

C8. SignaturePad
• 난이도: 높음
• 권장 기능
o 지우기
o “선이 그려졌는지” 감지(빈 서명 제출 방지)
• 대안
o 검증된 서명 라이브러리 사용(웹용 signature pad 계열)

C9. ExpandableSection(아코디언)
• 역할: 권장/선택 영역을 접어서 단순화
• 난이도: 낮음
• 주의: 펼침 상태 기억(세션 내) 정도만

C10. PhotoUploadSection
• 역할: 전/후 사진 업로드, 썸네일, 삭제
• 난이도: 중간~높음
• 리스크 포인트
o 모바일에서 큰 이미지 업로드 성능/실패
o 업로드 중 이탈/재시도 UX
• 필수 규칙(Phase 1)
o “권장”이므로 완료 제출 조건에는 기본 포함 X
o 업로드 실패 시 재시도 제공
• 구현 팁
o 이미지 리사이즈/압축(클라이언트 or 서버)
o 업로드 상태 표시(업로드 중…/완료/실패)

C11. CustomerNotifySection (선택)
• 역할: 문자/이메일 발송 옵션 설정
• 난이도: 낮음~중간
• 주의
o 번호/이메일 유효성 체크
o 현장에서는 “수정 가능”이 오히려 혼란일 수 있으니 정책으로 잠글 수 있게

C12. StickyActionBar
• 역할: 완료 제출 버튼 고정
• 난이도: 낮음
• 필수 규칙
o 검증 실패 시 해당 섹션으로 스크롤 이동
o 중복 제출 방지(연타 방지)

C13. Toast/AlertSystem
• 역할: 저장/오류/검증 결과 피드백
• 난이도: 낮음
• 필수 메시지
o 필수 누락
o 서명 누락
o 저장 실패
o 완료 성공/실패

C14. ConfirmModal (선택)
• 역할: 완료 제출 전 확인(실수 방지)
• 난이도: 낮음
• 권장
o 필드테스트 초반엔 OFF 가능(클릭 한 번 줄이기)

2) 기능 난이도 “핵심 3대 리스크”
실제 개발에서 시간 잡아먹는 TOP 3는 대개 이거야:
1. 서명 패드 품질/호환성
2. 사진 업로드 안정성(압축/재시도/네트워크)
3. 자동 저장 + 오류 복구 UX
이 3개가 안정적이면, 나머지는 CRUD라 비교적 빠르게 끝남.

3) 완료 제출 서버 검증(필수 명세)
완료 제출 API는 반드시 서버에서 최종 검증해야 함(클라만 믿으면 누락 생김).
• required checklist 전체 입력 여부
• FAIL 항목 note 존재 여부
• signature 존재 여부 + signer name 존재
• (정책) notify 체크 시 연락처 형식 검증
• 성공 시:
o WorkOrder status=COMPLETED
o completed_at set
o PDF 생성/발송 잡 큐(또는 비동기 트리거) 실행
o AuditLog 남김

4) Phase 1 구현 우선순위(권장)
필드테스트 빨리 굴리려면 순서를 이렇게:
1. ChecklistSection + 저장
2. SignatureSection + 저장
3. 완료 제출 + 서버 검증
4. 사진 업로드
5. 발송 옵션 섹션
6. 작업 시작/진행중 상태(선택)

(본문 끝)
===

## 연관 문서
- (추가 예정)

## 변경 이력
- v0.1: 통합문서 기반 초안 정리
