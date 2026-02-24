# 🏷️ Issue Labels Guide

이 프로젝트에서 사용하는 이슈 라벨 정의입니다.  
이슈를 생성/관리할 때 아래 규칙에 따라 적절한 라벨을 부착해 주세요.

> 참고: GitHub Projects의 **필드(Type/Gate/Area/Priority)** 를 함께 사용합니다.  
> - **필드**: 보드/로드맵에서 정렬·필터링용(권장: 항상 입력)  
> - **라벨**: 검색/공유/강조용(빠르게 파악)

---

## 1) 유형 (Type)

| 라벨명 | 설명 |
| :---: | :--- |
| `epic` | 여러 티켓을 포함하는 상위 작업 단위(Feature/큰 줄기). 보통 Gate 단위 목표를 담음 |
| `ticket` | 개별적으로 수행 가능한 최소 작업 단위(구현/수정/테스트/문서 등) |

---

## 2) 도메인 / 레이어 (Domain/Layer)

| 라벨명 | 설명 |
| :---: | :--- |
| `backend` | 서버 비즈니스 로직, DB, API |
| `frontend` | UI/UX, 클라이언트 로직(웹/모바일 웹) |
| `infra` | CI/CD, 배포, 환경설정, 스토리지/클라우드 리소스 |
| `docs` | 문서 작성/수정(README, spec, API 문서 등) |
| `qa` | 테스트 계획/수행, 버그 리포트 |

---

## 3) 우선순위 (Priority)

| 라벨명 | 우선순위 | 설명 |
| :---: | :---: | :--- |
| `P0-blocker` | 최고 | 막히면 다음 진행이 불가. 즉시 해결 필요 |
| `P1-core` | 높음 | Phase 1 핵심 기능. 일정에 필수 |
| `P2-nice` | 보통 | 있으면 좋음. 일정에 따라 조정 가능 |

---

## 4) 상태 / 리스크 (Status/Risk)

| 라벨명 | 설명 |
| :---: | :--- |
| `needs-review` | 작업 완료 후 리뷰/피드백이 필요한 상태(코드/문서/테스트) |
| `risk` | 일정 지연/기술적 부채/불확실성이 커서 주의가 필요한 상태 |
| `blocked` | 외부 요인 또는 의존성 때문에 진행이 멈춘 상태 |

---

## 5) 게이트 (Roadmap Gate)

| 라벨명 | 단계 | 설명 |
| :---: | :---: | :--- |
| `G1-foundation` | Gate 1 | 기반(Auth/RBAC/Org-Team-User) |
| `G2-workorder-core` | Gate 2 | WorkOrder 핵심 로직(배정 체인/목록/상세) |
| `G3-field-execution` | Gate 3 | 현장 수행(M-01/M-02, 제출 DoD) |
| `G4-evidence-ops` | Gate 4 | 증빙/PDF + 운영지표(대시보드) |

---

## 라벨 부착 기본 규칙(권장)

- 모든 이슈는 최소 3개 라벨을 권장:
  1) `epic` 또는 `ticket`  
  2) 도메인 1개(`backend`/`frontend`/`infra`/`docs`/`qa`)  
  3) 우선순위 1개(`P0-blocker`/`P1-core`/`P2-nice`)  
- Gate 라벨(`G1~G4`)은 해당 시점의 로드맵 분류가 필요할 때 추가(권장).