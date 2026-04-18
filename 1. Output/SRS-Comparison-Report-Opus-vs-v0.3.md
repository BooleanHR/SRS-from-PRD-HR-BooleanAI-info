# SRS 비교 분석 보고서
## Opus output-HR-AI-Verification (v1.0) vs SRS-HR-AI-Verification-v0.3

**작성일:** 2026-04-17  
**목적:** 두 SRS 문서 간 변경사항을 세 가지 관점으로 분류·정리하여, 후속 AI 에이전트 Task 추출의 기준선으로 활용

---

## 1) 기술 스택의 명확성

> Opus 문서는 기술 스택을 **암시적으로 전제**했으나, v0.3은 **C-TEC 시리즈로 명문화**하여 AI 코드 생성 시 스택 이탈을 원천 차단함.

| # | 비교 항목 | Opus output (v1.0) | SRS v0.3 | 변경 성격 |
|---|---|---|---|---|
| T-01 | **기술 스택 정의 섹션** | 없음. 아키텍처를 서술적으로 기술하되 구체적 프레임워크/라이브러리 미지정 | **섹션 2. Tech Stack Constraints (C-TEC-001 ~ 007)** 신설. Next.js App Router, Prisma, Supabase, Vercel AI SDK, shadcn/ui, Tailwind CSS, pdf-lib 등 고정 | 🟢 신규 추가 |
| T-02 | **프론트엔드 프레임워크** | 미지정 ("Web UI", "Web Dashboard"로만 표현) | **Next.js App Router** 단일 풀스택 의무화 (C-TEC-001). 별도 Express/FastAPI/NestJS 금지 | 🟢 명확화 |
| T-03 | **백엔드 로직 구현 방식** | "검증 엔진", "API Gateway" 등 추상적 컴포넌트 | **Server Actions + Route Handlers** 의무화 (C-TEC-002). `"use server"` 지시어 활용 | 🟢 명확화 |
| T-04 | **DB 기술** | "Database"로만 표현. ORM, DB 엔진 미지정 | **Prisma + SQLite(로컬) → Supabase PostgreSQL(배포)** 이중 전략 (C-TEC-003). JSON 타입 → Text 대체로 SQLite 호환성 확보 | 🟢 명확화 |
| T-05 | **UI 프레임워크** | 미지정 | **Tailwind CSS + shadcn/ui** 의무화 (C-TEC-004). CSS 파일 직접 작성 금지 | 🟢 명확화 |
| T-06 | **AI/LLM 호출 방식** | "OCR 엔진"으로 추상화. 구체적 SDK 미지정 | **Vercel AI SDK** 의무화 (C-TEC-005). `generateObject()` / `streamText()` API 명시 | 🟢 명확화 |
| T-07 | **LLM 모델** | 미지정 (범용 OCR 전제) | **Google Gemini API** 고정 (C-TEC-006). `GEMINI_API_KEY` 환경변수 | 🟢 명확화 |
| T-08 | **배포 인프라** | 미지정 (일반적 클라우드 전제) | **Vercel** 단일화 (C-TEC-007). `git push` 배포, Vercel 환경변수 패널 | 🟢 명확화 |
| T-09 | **인증 시스템** | "API Key + JWT 이중 인증" (REQ-NF-026) | **Supabase Auth** (이메일/패스워드 or Google OAuth) + **RLS** 기반 RBAC | 🔴 대폭 변경 |
| T-10 | **PDF 생성 라이브러리** | 미지정 ("PDF 동기 생성"으로만 표현) | **pdf-lib** 명시. 순수 JS, Vercel 호환 | 🟢 명확화 |
| T-11 | **이메일 발송** | 미지정 ("이메일 서비스"로 추상화) | **Resend API** 명시. 카카오 알림톡 완전 대체 | 🟢 명확화 |
| T-12 | **실시간 업데이트** | 미지정 (실시간 대시보드 전제) | **SWR 5초 polling** 명시 (ADR-007). Supabase Realtime/WebSocket 제거 | 🔴 대폭 변경 |
| T-13 | **Prisma 스키마** | 없음. 엔터티 필드를 표 형식으로만 정의 | **완전한 Prisma schema.prisma 코드** 제공 (약 170줄). enum, relation, @@map 포함 | 🟢 신규 추가 |
| T-14 | **스택 선택 이유 · 제한사항** | 없음 | **섹션 2.3** 신설. 각 스택별 선택 이유 + 알려진 제한사항 표 | 🟢 신규 추가 |
| T-15 | **월 운영 비용 예상** | 없음 | **섹션 3.16** 신설. 서비스별 플랜·비용 표 ($0 ~ $5/월 PoC) | 🟢 신규 추가 |

---

## 2) MVP 목표 및 가치전달 조정 내용

> Opus 문서는 **상용 수준의 전체 제품 SRS**로 작성되었으나, v0.3은 **"입문자 수준 개발자가 바이브코딩으로 구현 가능한 MVP PoC"**로 범위를 대폭 축소·재조정함.

| # | 비교 항목 | Opus output (v1.0) | SRS v0.3 | 변경 성격 |
|---|---|---|---|---|
| M-01 | **문서 성격** | 상용 제품 전체 SRS (Phase 1 ~ 3 포함) | **MVP PoC 기준 SRS**. "입문자 수준 개발자가 바이브코딩으로 실제 구현 가능한 범위" 명시 | 🔴 근본 변경 |
| M-02 | **PoC 증명 요소** | 없음 (전체 제품 관점) | **섹션 1.1** 신설. 6개 핵심 증명 항목 명시 (OCR 가능성, API 호출 성공, 불일치 탐지, Audit Trail 생성, 승인/반려, PDF 다운로드) | 🟢 신규 추가 |
| M-03 | **Mock/Dummy 대체 목록** | 없음 (모든 기능 실제 구현 전제) | **섹션 1.2** 신설. 6개 항목 Mock 대체: RPA, HRDK API, 카카오 알림톡, 병렬 캡처, 배치 처리, Supabase Realtime | 🟢 신규 추가 |
| M-04 | **Phase 2+ 제외 항목** | Out-of-Scope이 Phase 3만 명시 | **섹션 1.3** 신설. REST API 외부 연동, ORGANIZATION 멀티테넌시, ISMS-P 등 6개 항목 Phase 2+ 제외 | 🟢 신규 추가 |
| M-05 | **RPA 기반 기관 조회** | Must 기능 (REQ-FUNC-009 ~ 011). RPA 실제 구현 + 기관 사이트 변경 감지 + Slack 알림 | **완전 제거**. `mockAgencyResponse()` 함수로 대체. Vercel에서 Headless Browser 실행 불가 명시 | 🔴 제거 |
| M-06 | **카카오 알림톡 Self-Service** | Should 기능 (REQ-FUNC-029 ~ 036). 8개 세부 요건. 발송→재제출→재검증 전체 루프 | **완전 제거**. 이메일(Resend API) + `console.log` + 대시보드 텍스트 표시로 대체 | 🔴 제거 |
| M-07 | **REST API 외부 연동** | Could 기능 (REQ-FUNC-039 ~ 044). ATS SDK, 개발자 포털, Swagger, Sandbox | **완전 제거** (Phase 2) | 🔴 제거 |
| M-08 | **배치 처리 스케줄러** | Could 기능 (REQ-FUNC-045 ~ 046). 4,000건/시간 병렬 처리 | **완전 제거** (Phase 2). 단일 건 처리만 구현 | 🔴 제거 |
| M-09 | **병렬 캡처 (Parallel Capture)** | Must 핵심 기능 (REQ-FUNC-012 ~ 013). 원본+조회본 단일 프레임 캡처 + ±1초 타임스탬프 | **Placeholder 이미지 URL로 대체**. 원본 파일 경로만 저장. Browserless.io 제거 | 🔴 대폭 축소 |
| M-10 | **OCR 신뢰도 임계치** | 80 (CON-07) | **70으로 완화** (ADR-005). PoC 단계에서 이미지 품질 편차 고려 | 🟡 완화 |
| M-11 | **Audit Trail 불변성** | WORM 스토리지 + 물리적 불변성. 변조 시도 시 보안 알림 + 사용자 차단 (REQ-FUNC-015 ~ 017) | **논리적 불변성**으로 변경. `is_immutable=true` + 삭제 API 미노출 (ADR-006) | 🟡 축소 |
| M-12 | **Human-in-the-loop 우선순위** | Should (F6, REQ-FUNC-026 ~ 028) | **Must 승격** (F5, REQ-FUNC-030 ~ 033). PoC 핵심 흐름으로 격상 | 🟢 승격 |
| M-13 | **PDF 리포트 규모** | 500건 이하 동기(3초) / 500건 초과 비동기+이메일 | **100건 이하** 동기 처리(10초). 비동기 처리 제거 | 🟡 축소 |
| M-14 | **처리 SLA** | 단일 서류 E2E p95 ≤ 5초, 배치 4,000건/시간, PDF ≤ 3초 | E2E p95 ≤ 20초(OCR) + p95 ≤ 30초(Triple Check). **대폭 완화** | 🔴 대폭 완화 |
| M-15 | **가용성 SLA** | ≥ 99.5% (≤ 3.6시간/월 다운타임) | **≥ 99%** (무료 플랜 기준). 무료 플랜 다운타임 허용 | 🟡 완화 |
| M-16 | **오류율 SLA** | ≤ 0.5% | **≤ 2%** (PoC 완화) | 🟡 완화 |
| M-17 | **대상 사용자** | 6개 페르소나 (공공기관PM, 대행사팀장, HRBP, 법무관, ATS PO, 긴급채용PM) | **4개로 축소** (대행사 담당자, 팀장, 감사자, 개발자). 단일 조직 기준 | 🟡 축소 |
| M-18 | **이미지 전처리** | Should 기능 (REQ-FUNC-037 ~ 038). DPI 보정, 이진화, 노이즈 제거, Deskew | **"Gemini Vision 내장 처리"**로 대체. 별도 전처리 모듈 미구현 | 🟡 축소 |
| M-19 | **기능 요구사항 수** | 47개 (REQ-FUNC-001 ~ 047) | **약 20개**로 축소 (REQ-FUNC-001 ~ 061, 번호 재배정) | 🔴 대폭 축소 |
| M-20 | **비기능 요구사항 수** | 49개 (REQ-NF-001 ~ 049) | **약 15개**로 축소 | 🔴 대폭 축소 |

---

## 3) 기타 차이점

> 문서 구조, 다이어그램, 가이드, 검증 계획 등 위 두 카테고리에 포함되지 않는 차이점.

| # | 비교 항목 | Opus output (v1.0) | SRS v0.3 | 변경 성격 |
|---|---|---|---|---|
| E-01 | **문서 구조** | 표준 SRS 7장 구조 (Introduction, Stakeholders, Context, Requirements, Traceability, Appendix) | **3.x 번호 체계**로 재구성. PoC Scope → Tech Stack → Introduction → ERD → Sequence → Component → Use Case → Class → Requirements → NFR → RTM → 가이드 → 검증 계획 | 🟡 재구성 |
| E-02 | **ERD 엔터티** | 6개 (APPLICANT, DOCUMENT, VERIFICATION_JOB, AUDIT_TRAIL, NOTIFICATION, REPORT) | **8개** (USER, BATCH 추가). USER에 role/passwordHash/mfaEnabled 포함. BATCH로 채용 회차 관리 | 🟢 확장 |
| E-03 | **ERD 관계** | ORGANIZATION → APPLICANT 다중테넌시 포함 | ORGANIZATION **제거**. 단일 조직 기준. USER → BATCH → APPLICANT 구조로 변경 | 🔴 변경 |
| E-04 | **Stakeholders 섹션** | **섹션 2** (9개 역할, 상세 페르소나+관심사) | **제거**. User Characteristics로 간소화 (4개 사용자 유형) | 🔴 제거 |
| E-05 | **변경 이력** | 없음 | **Revision History 테이블** 신설 (v0.2-Opus → v0.2-Sonnet → v0.3) | 🟢 신규 추가 |
| E-06 | **v0.3 변경 핵심 원칙** | 없음 | **경고 박스** 신설. 제거된 것 / 추가된 것 명시 | 🟢 신규 추가 |
| E-07 | **시퀀스 다이어그램** | 4개 (핵심 Triple Check, PDF 생성, Human-in-the-loop, 상세 전체 흐름) + 3개 부록 (Self-Service, ATS 연동, Audit Trail 불변성) | **3개** (서류 업로드→Triple Check→Audit Trail, 승인/반려, PDF 생성). 모두 MVP 단순화 버전. Next.js Server Action / Supabase 구체적 등장 | 🟡 축소+구체화 |
| E-08 | **컴포넌트 다이어그램** | 없음 (서술적 기술) | **Mermaid 그래프** 신설. 11개 컴포넌트별 역할 정의표 포함 | 🟢 신규 추가 |
| E-09 | **Use Case 다이어그램** | 없음 | **Mermaid 그래프** 신설. 3개 Actor × 9개 Use Case | 🟢 신규 추가 |
| E-10 | **Class 다이어그램** | 없음 | **Mermaid 클래스 다이어그램** 신설. 7개 서비스 클래스, 메서드·의존관계 명시 | 🟢 신규 추가 |
| E-11 | **구현 난이도 가이드** | 없음 | **섹션 3.16** 신설. 12단계 구현 순서, 단계별 예상 소요·난이도(⭐ ~ ⭐⭐⭐), 총 2 ~ 3주 예상 | 🟢 신규 추가 |
| E-12 | **AI 프롬프트 작성 가이드** | 없음 | **DO / DON'T 가이드** 신설. 바이브코딩 시 올바른 지시 vs 범위 초과 지시 예시 | 🟢 신규 추가 |
| E-13 | **Validation Plan** | 3개 실험 (병렬 캡처 면책권 A/B테스트, Self-Service 민원 감소 Before-After, 처리 속도 PoC) — 통계적 검증 설계 | **5개 실험** (OCR 정확도 10건, FAIL 감지 5건, Audit Trail 불변성, PDF 완성도 10건, E2E 데모) — 실행 가능한 수동 검증 | 🔴 완전 재작성 |
| E-14 | **용어 정의 (Definitions)** | 17개 용어 (AOS, DOS, JTBD, Persona, MoSCoW 등 비즈니스 용어 포함) | **19개 용어**. 비즈니스 용어 제거, 기술 용어 추가 (Server Action, Route Handler, SWR, pdf-lib, Resend, Prisma, Supabase Auth, RLS 등) | 🟡 재작성 |
| E-15 | **API 엔드포인트** | 10개. REST API v1 체계 (`/api/v1/...`). API Key+JWT, Rate Limit 포함 | **5개**로 축소. Route Handler 체계 (`/api/...`). Supabase Session 인증. Server Action 우선 권장 | 🔴 대폭 축소 |
| E-16 | **오류 응답 표준** | 미지정 (개별 AC에서 HTTP 코드 언급) | **오류 응답 표준 표** 신설 (400, 401, 403, 404, 422, 500) | 🟢 신규 추가 |
| E-17 | **References** | 9개 (PRD, ISO, VPS, 감사원 보고서, 법률, 인권위, JTBD 인터뷰, 파트너 실사, PoC ROI) | **7개**로 축소. 기술 공식 문서 추가 (Next.js, Vercel AI SDK, Supabase, Prisma) | 🟡 재구성 |
| E-18 | **ADR (Architecture Decision Records)** | 5개 (ADR-001 ~ 005). RPA 복구 SLA, Human-in-the-loop, 공공API 쿼터, 개인정보보호, OCR 임계치 | **7개** (ADR-001 ~ 007). 단일 풀스택, API 우선 연동, Mock 대체, SWR polling 등 추가 | 🟡 확장+재작성 |
| E-19 | **RTM (Traceability Matrix)** | 4방향 (Story→REQ, Feature→REQ, KPI→NFR, Risk→Constraint) — 상세 | **정방향+역방향** 2방향. MVP 범위 내 기능만 추적. 간소화 | 🟡 축소 |
| E-20 | **비용 요구사항** | REQ-NF-034 ~ 035. 4,000건 ≤ 500만원, 건당 ≤ 1,250원 | **제거**. 월 운영 비용 $0 ~ $5 예상으로 대체 | 🔴 변경 |

---

## 종합 요약

| 관점 | Opus output (v1.0) | SRS v0.3 | 핵심 차이 |
|---|---|---|---|
| **기술 스택** | 미지정 (추상적) | C-TEC 시리즈로 완전 고정 | v0.3이 AI 코드 생성 직접 지시 가능 수준 |
| **MVP 범위** | 상용 제품 전체 (47 기능 + 49 비기능) | PoC 최소 범위 ( ~ 20 기능 +  ~ 15 비기능) | v0.3이 "실제 구현 가능"에 초점 |
| **가치 전달** | 비즈니스 KPI 중심 (인건비 절감, 민원 감소) | 기술 증명 중심 (OCR 가능성, API 호출, Audit Trail) | v0.3이 "증명 가능한 것"에 집중 |
| **대상 독자** | 개발팀·QA·PM·법무·보안·감사자 | **입문자 개발자 + AI 에이전트** | v0.3이 "바이브코딩 지시서"로 직접 활용 가능 |

---

*— End of Comparison Report —*
