# SRS 수용기준(Definition of Done) 검토 리포트

**검토 대상:** `Opus_output-HR-AI-Verification.md` (SRS-001 v1.0)
**검토 기준:** SRS DoD 체크리스트 (ISO/IEC/IEEE 29148:2018 기반)
**검토 일자:** 2026-04-15
**검토자:** Claude Sonnet 4.6 (AI Review)

---

## 종합 판정

| 기준 항목 | 판정 | 세부 결과 |
|---|---|---|
| ① PRD 모든 Story·AC → REQ-FUNC 반영 | ✅ 충족 | Story-1~4 및 각 AC 전량 REQ-FUNC에 Source 컬럼으로 추적 완료 |
| ② 모든 KPI·성능 목표 → REQ-NF 반영 | ✅ 충족 | 북극성 KPI 포함 KPI-1~5 전량 NFR 매핑 완료 |
| ③ API 목록 → 인터페이스 섹션 완전 반영 | ⚠️ 부분 미충족 | `GET /dashboard/stats` (A-07)이 §3.3에서 누락 |
| ④ 엔터티·스키마 → Appendix 완성 | ⚠️ 부분 미충족 | 엔터티 테이블은 완비, mermaid ERD 다이어그램 없음 |
| ⑤ Traceability Matrix 누락 없이 생성 | ⚠️ 부분 미충족 | REQ-FUNC-037, 038, 045~047의 Story 역방향 추적 누락 |
| ⑥ 핵심 다이어그램 전종 작성 (UC·ERD·Class·Component) | ❌ 미충족 | Use Case·ERD·Class·Component Diagram 4종 모두 없음 |
| ⑦ Sequence Diagram 3~5개 포함 | ⚠️ 기준 초과 | 총 7개 포함 (상한 5개 초과) |
| ⑧ SRS 전체 ISO 29148 구조 준수 | ⚠️ 부분 준수 | 전반적 구조 준수, Overall Description 섹션 부재 |

**DoD 충족률: 2/8 완전 충족 · 4/8 부분 미충족 · 1/8 기준 초과 · 1/8 미충족**

---

## 항목별 상세 검토

---

### ① PRD의 모든 Story·AC가 SRS의 REQ-FUNC에 반영됨 ✅ 충족

**근거:**

| Story | 커버 REQ-FUNC | AC 추적 |
|---|---|---|
| Story-1 (Audit Trail 기반 법적 면책권) | REQ-FUNC-012~017, 023~025 | AC-1(§4.1.3), AC-2(§4.1.5), AC-3(§4.1.3) ✅ |
| Story-2 (Triple Check Loop 무오성 검증) | REQ-FUNC-001~011, 018~022, 026~028 | AC-1(§4.1.4), AC-2(§4.1.2), AC-3(§4.1.4) ✅ |
| Story-3 (Self-Service 알림톡 루프) | REQ-FUNC-029~036 | AC-1(§4.1.7), AC-2(§4.1.7), AC-3(§4.1.7) ✅ |
| Story-4 (API First 플랫폼 연동) | REQ-FUNC-039~044 | AC-1(§4.1.9), AC-2(§4.1.9), AC-3(§4.1.9) ✅ |

- 모든 REQ-FUNC에 `Source` 컬럼이 있어 PRD Story·AC 역추적 가능
- Acceptance Criteria가 Given-When-Then 형식으로 명확히 기술됨
- **특이사항 없음**

---

### ② 모든 KPI·성능 목표가 REQ-NF에 반영됨 ✅ 충족

**§5.3 KPI → NFR Mapping 기준 검토:**

| KPI | NFR 매핑 | 비고 |
|---|---|---|
| 북극성 KPI (Audit Trail 자동 생성율 ≥ 99%) | REQ-NF-022 ✅ | 정확도 섹션에 반영 |
| KPI-1 (처리속도 ≥ 4,000건/시간) | REQ-NF-005, NF-006, NF-042 ✅ | 성능+확장성 이중 반영 |
| KPI-2 (인건비 ≤ 500만원) | REQ-NF-034, NF-035 ✅ | 비용 섹션 별도 정의 |
| KPI-3 (민원 전화 감소 ≥ 90%) | REQ-NF-048, NF-049 ✅ | Operational Efficiency 섹션 |
| KPI-4 (허위 검출 정확도 ≥ 95%) | REQ-NF-019, NF-020 ✅ | False Negative 별도 정의 |
| KPI-5 (감사 리포트 자동 생성률 100%) | REQ-NF-023 ✅ | 정확도 섹션 |

- 9개 성능 지표(REQ-NF-001~009)에 측정 지표·목표값·측정방법·Source 4열 완비
- **특이사항 없음**

---

### ③ API 목록이 인터페이스 섹션에 모두 반영됨 ⚠️ 부분 미충족

**§3.3 (API Overview) vs §6.1 (Appendix API Endpoint List) 비교:**

| # | Endpoint | §3.3 | §6.1 | 상태 |
|---|---|---|---|---|
| A-01 | `POST /api/v1/verification/submit` | ✅ | ✅ | 정상 |
| A-02 | `GET /api/v1/verification/{job_id}` | ✅ | ✅ | 정상 |
| A-03 | `POST /api/v1/batch/submit` | ✅ | ✅ | 정상 |
| A-04 | `GET /api/v1/batch/{batch_id}/status` | ✅ | ✅ | 정상 |
| A-05 | `GET /api/v1/report/{batch_id}/pdf` | ✅ | ✅ | 정상 |
| A-06 | `POST /api/v1/notification/send` | ✅ | ✅ | 정상 |
| **A-07** | **`GET /api/v1/dashboard/stats`** | **❌ 누락** | ✅ | **⚠️ 불일치** |
| A-08 | 정부24 진위확인 API | ✅ | ✅ | 정상 |
| A-09 | HRDK API | ✅ | ✅ | 정상 |
| A-10 | 카카오 알림톡 API | ✅ | ✅ | 정상 |

**결함:** `GET /api/v1/dashboard/stats`(A-07)가 Appendix §6.1에는 존재하나 §3.3 API Overview 테이블에서 누락됨.

**조치 권고:** §3.3 API Overview에 A-07 행 추가. 입력/출력 파라미터, 인증 방식, 제약사항 기술 필요.

---

### ④ 엔터티·스키마가 Appendix에 완성됨 ⚠️ 부분 미충족

**§6.2 엔터티 정의 현황:**

| 엔터티 | 필드 정의 | 제약조건 | PK/FK 관계 |
|---|---|---|---|
| APPLICANT | ✅ 5개 필드 | ✅ 완비 | ✅ |
| DOCUMENT | ✅ 7개 필드 | ✅ 완비 | ✅ FK→APPLICANT |
| VERIFICATION_JOB | ✅ 11개 필드 | ✅ 완비 | ✅ FK→DOCUMENT |
| AUDIT_TRAIL | ✅ 7개 필드 | ✅ 완비 | ✅ FK→VERIFICATION_JOB |
| NOTIFICATION | ✅ 7개 필드 | ✅ 완비 | ✅ FK→APPLICANT |
| REPORT | ✅ 6개 필드 | ✅ 완비 | ✅ FK→batch_id |

**결함:** §6.2.7에 Entity Relationship Summary가 텍스트 테이블(5개 관계 기술)로만 존재하며, **mermaid 형식의 ERD 다이어그램이 없음.** 시각적 관계 파악이 불가하고, DoD 기준의 "다이어그램" 요건 미충족.

**추가 결함:** REPORT 엔터티의 `batch_id` FK가 어느 테이블을 참조하는지 명시되지 않음 (타겟 테이블 불명확).

**조치 권고:**
1. mermaid `erDiagram` 문법으로 6개 엔터티 ERD 작성
2. REPORT.batch_id의 FK 참조 테이블 명시 (신규 BATCH 엔터티 추가 또는 VERIFICATION_JOB 내 batch_id 컬럼 정의 필요)

---

### ⑤ Traceability Matrix가 누락 없이 생성됨 ⚠️ 부분 미충족

**§5.1 Story→REQ 매핑에서 역추적 불가 요구사항:**

| REQ-FUNC | 기능 | §5.1 Story 매핑 | 상태 |
|---|---|---|---|
| REQ-FUNC-037 | 저화질 이미지 전처리 (DPI < 150) | ❌ 미매핑 | **누락** |
| REQ-FUNC-038 | 기울어진 이미지 Deskew | ❌ 미매핑 | **누락** |
| REQ-FUNC-045 | 배치 처리 스케줄러 | ❌ 미매핑 | **누락** |
| REQ-FUNC-046 | 배치 처리 진행 상태 표시 | ❌ 미매핑 | **누락** |
| REQ-FUNC-047 | 불일치 신뢰도 점수 시각화 대시보드 | ❌ 미매핑 | **누락** |

- §5.2 Feature→REQ 매핑(F8·C2·C3)은 완비되어 있으나, §5.1 Story 역방향 매핑 5건 누락
- REQ-NF-036~041 (운영·모니터링 NFR) 및 REQ-NF-042~044 (확장성 NFR)에 대응하는 Test Case ID가 §5.1에 미정의

**조치 권고:**
1. §5.1에 F8(S3)·C2·C3에 해당하는 Feature 단위 행 추가 또는 관련 Story와 연결
2. REQ-NF 전체에 대응하는 테스트 케이스 ID를 §5.1 또는 별도 NFR Test Matrix로 정의

---

### ⑥ 핵심 다이어그램 전종 작성 ❌ 미충족

**다이어그램 존재 여부:**

| 다이어그램 종류 | 요건 | 실제 현황 | 판정 |
|---|---|---|---|
| **Use Case Diagram** (mermaid) | 필수 | **없음** | ❌ |
| **ERD** (mermaid erDiagram) | 필수 | 없음 (텍스트 테이블만) | ❌ |
| **Class Diagram** (mermaid classDiagram) | 필수 | **없음** | ❌ |
| **Component Diagram** (mermaid 또는 PlantUML) | 필수 | **없음** | ❌ |
| Sequence Diagram | 포함 | 7개 (§3.4 + §6.3) | ✅ |

**결함 요약:** Sequence Diagram 7개는 충분히 작성되었으나, Use Case·ERD·Class·Component Diagram 4종이 전무하여 DoD 핵심 다이어그램 기준 완전 미충족.

**조치 권고 (우선순위 순):**

1. **Use Case Diagram** — 주요 Actor(지원자·담당자·관리자·감사자·ATS 플랫폼)와 Use Case(서류 업로드·진위확인·Audit Trail 열람·PDF 다운로드·승인/반려 등) 관계를 mermaid 형식으로 작성

2. **ERD** — §6.2.7의 관계 요약을 mermaid `erDiagram`으로 시각화 (상기 ④ 항목과 연계)

3. **Class Diagram** — VerificationEngine, OCREngine, AuditTrailService, NotificationService, ReportService 등 핵심 서비스 클래스와 메서드 시그니처 정의

4. **Component Diagram** — Web UI / API Gateway / Verification Engine / OCR Engine / RPA Module / Public API Connector / Audit Trail Service / DB / External APIs 간 의존 관계 및 인터페이스 정의

---

### ⑦ Sequence Diagram 3~5개 포함됨 ⚠️ 기준 초과 (실질 충족)

**포함된 Sequence Diagram 목록:**

| ID | 위치 | 주제 |
|---|---|---|
| SD-01 | §3.4.1 | 단일 서류 Triple Check Loop 검증 |
| SD-02 | §3.4.2 | 감사 리포트 PDF 생성 |
| SD-03 | §3.4.3 | Human-in-the-loop 최종 승인 |
| SD-04 | §6.3.1 | 서류 업로드→OCR→Triple Check→Audit Trail 전체 흐름 |
| SD-05 | §6.3.2 | Self-Service 알림톡 루프 |
| SD-06 | §6.3.3 | 외부 ATS 플랫폼 API 연동 |
| SD-07 | §6.3.4 | Audit Trail 불변성 보장 및 접근 로그 |

DoD 기준 "3~5개"를 초과하여 7개 포함. 내용상으로는 시스템 핵심 흐름을 모두 커버하고 있어 실질적으로 충분. 단, SD-01과 SD-04가 동일 흐름의 축약/상세 버전으로 중복성이 있음.

**조치 권고:** SD-01을 개요용(§3 수준)으로 유지하고, SD-04를 상세 설계 다이어그램(§6 수준)으로 명시적으로 구분 표기하면 중복 우려 해소 가능.

---

### ⑧ SRS 전체 ISO 29148 구조 준수 ⚠️ 부분 준수

**ISO/IEC/IEEE 29148:2018 필수 섹션 대비 현황:**

| ISO 29148 섹션 | 요건 | SRS 현황 | 판정 |
|---|---|---|---|
| 1. Introduction (Purpose·Scope·Definitions·References) | 필수 | §1.1~1.4 완비 | ✅ |
| 2. Overall Description (System perspective·Product functions·User characteristics·Constraints·Assumptions·Dependencies) | 필수 | **독립 섹션 없음.** Constraints(§1.2.4)·Assumptions(§1.2.5)는 Scope 하위에, User characteristics는 Stakeholders(§2)로 별도 분리됨 | ⚠️ |
| 3. System Context and Interfaces | 필수 | §3 완비 (External Systems·Client Apps·API Overview·Sequence Diagrams) | ✅ |
| 4. Specific Requirements (Functional·Non-Functional) | 필수 | §4.1 (REQ-FUNC), §4.2 (REQ-NF) 완비 | ✅ |
| 5. Traceability | 권장 | §5 (4종 Matrix) 완비 | ✅ |
| 6. Appendix | 권장 | §6 (API·엔터티·Sequence·검증계획) 완비 | ✅ |

**결함:** ISO 29148의 Section 2 "Overall Description"에 해당하는 독립 섹션이 없음. Stakeholders가 Section 2로 배치되어 있으나, 이는 표준의 "Product functions 개요"·"User characteristics"·"Constraints/Assumptions 종합"을 포괄하는 Overall Description과 구조적으로 다름.

**조치 권고:** 현행 §2(Stakeholders)를 "§2. Overall Description"으로 확장하고, 아래 하위 항목 추가:
- §2.1 System Perspective (시스템 맥락 및 동작 원리 개요)
- §2.2 Product Functions (기능 요약 목록)
- §2.3 User Characteristics (현행 Stakeholders 재배치)
- §2.4 Constraints, Assumptions, Dependencies (현행 §1.2.4~1.2.6 이동)

---

## 수정 우선순위 요약

| 우선순위 | 항목 | 작업 내용 |
|---|---|---|
| **P1 (즉시)** | 핵심 다이어그램 4종 추가 | mermaid Use Case·ERD·Class·Component Diagram 신규 작성 |
| **P1 (즉시)** | Traceability Matrix 보완 | REQ-FUNC-037, 038, 045~047 Story 매핑 추가 |
| **P2 (단기)** | A-07 API §3.3 반영 | `GET /dashboard/stats` 행 §3.3에 추가 |
| **P2 (단기)** | ERD mermaid 다이어그램화 | §6.2.7 텍스트 관계표를 mermaid `erDiagram`으로 변환 |
| **P3 (중기)** | ISO 29148 §2 구조 개선 | Overall Description 섹션 신설 및 Stakeholders 재구성 |
| **P3 (중기)** | REPORT FK 명시 | `batch_id` 참조 테이블 또는 BATCH 엔터티 정의 |

---

## 결론

SRS-001은 기능 요구사항(REQ-FUNC 1~47), 비기능 요구사항(REQ-NF 1~49), 트레이서빌리티 4종 매트릭스, Sequence Diagram 7개 등 핵심 콘텐츠의 완성도는 높다. 특히 Given-When-Then 형식의 Acceptance Criteria, KPI→NFR 역추적, 엔터티 스키마 정의는 산업 수준 이상으로 작성되어 있다.

그러나 **DoD 관점에서 Use Case·ERD(mermaid)·Class·Component Diagram 4종 다이어그램이 전무하여 설계 가시성이 부족**하며, 이것이 현재 가장 큰 미충족 항목이다. P1 항목 2건(다이어그램 4종 + Traceability 보완)을 완료하면 DoD 기준을 실질적으로 충족할 수 있다.

---
*본 리뷰는 SRS-001 v1.0 전체 946줄을 대상으로 DoD 8개 기준에 따라 수행되었습니다.*
