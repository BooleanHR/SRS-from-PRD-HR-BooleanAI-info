# SRS Revision Recipe v0.3
## AI 에이전트 오케스트레이팅 작업 지시서

**문서 목적:** SRS v0.3 기반 MVP PoC를 단계별로 AI 에이전트에게 지시하여 구현하기 위한 작업 레시피  
**기준 문서:** `SRS-HR-AI-Verification-v0.3.md` (Revision 0.3, 2026-04-16)  
**대상 AI:** Claude, Cursor, Windsurf 등 코드 생성 AI 에이전트  
**운영 방식:** 각 Phase를 순차 실행. 이전 Phase의 검증 완료 후 다음 Phase 진입  
**예상 총 소요:** 2~3주 (바이브코딩 기준)

---

## 문서 구조 개요

```
Recipe 구조:

Phase 0  ── 프로젝트 초기화 (Scaffolding)
Phase 1  ── 데이터 기반 구축 (Schema + Auth)
Phase 2  ── 핵심 엔진 구현 (OCR + Triple Check)
Phase 3  ── 감사 체계 구현 (Audit Trail + 불변성)
Phase 4  ── 사용자 인터페이스 (Dashboard + 승인/반려)
Phase 5  ── 출력물 생성 (PDF 리포트)
Phase 6  ── 알림 체계 (이메일 알림)
Phase 7  ── 통합 테스트 + 배포

각 Phase 내부 구조:
├── 목표 (Objective)
├── 선행 조건 (Prerequisites)
├── 연계 SRS 요구사항 (REQ-FUNC / REQ-NF)
├── AI 프롬프트 (Prompt) — 실제 지시문
├── 생성 산출물 (Expected Output)
├── 검증 체크리스트 (Verification)
└── 주의사항 / 금지 패턴 (Guard Rails)
```

---

## 공통 시스템 프롬프트 (모든 Phase에 선행 주입)

> 아래 블록은 모든 Phase의 AI 프롬프트 앞에 **시스템 컨텍스트**로 주입한다.  
> Phase별 프롬프트는 이 컨텍스트를 전제로 작성되어 있다.

```markdown
## System Context — HR AI 서류 진위확인 솔루션 MVP

### 기술 스택 제약 (C-TEC)
- C-TEC-001: Next.js App Router 단일 풀스택. 별도 Express/FastAPI/NestJS 서버 금지
- C-TEC-002: 서버 로직 = Server Actions (`"use server"`) 또는 Route Handlers (`app/api/`)만 사용
- C-TEC-003: DB = Prisma ORM. 로컬 SQLite(개발) → Supabase PostgreSQL(배포). JSON 컬럼은 Text 타입으로 통일
- C-TEC-004: UI = Tailwind CSS + shadcn/ui만 사용. CSS 파일 직접 작성 금지
- C-TEC-005: AI/LLM = Vercel AI SDK (`generateObject()` 또는 `streamText()`). 별도 Python 서버 금지
- C-TEC-006: LLM = Google Gemini API. 환경변수 `GEMINI_API_KEY`
- C-TEC-007: 배포 = Vercel. `git push`만으로 배포. 시크릿은 Vercel 환경변수 패널

### 금지 패턴 (DON'T)
- Inngest / QStash / BullMQ 등 외부 작업 큐
- Playwright / Puppeteer / Browserless.io 등 Headless Browser
- Supabase Realtime WebSocket (→ SWR 5초 polling 사용)
- Redis / Kafka 등 메시지 브로커
- Docker Compose / K8s 등 컨테이너 오케스트레이션
- CSS Modules / Styled Components / Emotion 등 CSS-in-JS

### 핵심 설계 원칙
- ADR-004: Human-in-the-loop 의무화 — AI 결과는 참고, 최종 승인은 인간
- ADR-005: OCR 신뢰도 임계치 = 70 (PoC 기준)
- ADR-006: 논리적 불변성 — `is_immutable=true` + 삭제 API 미노출
- ADR-007: SWR 5초 polling으로 실시간성 대체
- Mock 사용 시 반드시 `mock_used=true` 플래그 기록
```

---
---

## Phase 0 — 프로젝트 초기화 (Scaffolding)

### 목표
Next.js App Router 프로젝트 생성, shadcn/ui 설치, Supabase 연결, 기본 폴더 구조 확립

### 선행 조건
- Node.js 18+ 설치
- npm/pnpm 설치
- Supabase 프로젝트 생성 완료 (무료 플랜)
- Google Gemini API 키 발급 완료

### 연계 SRS 요구사항
- C-TEC-001 ~ C-TEC-007 (기술 스택 전체)

### AI 프롬프트

```markdown
## Task: Next.js 프로젝트 초기화

### 요청
Next.js App Router 기반 프로젝트를 생성해줘. 아래 요구사항을 정확히 따라줘.

### 프로젝트 구조
```
hr-ai-verification/
├── app/
│   ├── (auth)/           # 인증 관련 페이지 그룹
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/      # 인증 필요 페이지 그룹
│   │   ├── dashboard/    # 메인 대시보드
│   │   ├── verifications/# 검증 목록 + 상세
│   │   ├── review/       # 승인/반려 큐
│   │   └── reports/      # PDF 리포트
│   ├── api/              # Route Handlers
│   │   ├── verifications/
│   │   ├── reports/
│   │   └── dashboard/
│   ├── layout.tsx
│   └── page.tsx          # 랜딩/리다이렉트
├── components/
│   └── ui/               # shadcn/ui 컴포넌트
├── lib/
│   ├── supabase/         # Supabase 클라이언트 설정
│   ├── gemini/           # Vercel AI SDK + Gemini 설정
│   ├── actions/          # Server Actions
│   └── utils/            # 유틸리티 함수
├── prisma/
│   └── schema.prisma
├── .env.local            # 로컬 환경변수
└── middleware.ts          # Auth 미들웨어
```

### 환경변수 (.env.local 템플릿)
```env
# Supabase
DATABASE_URL="file:./dev.db"
DIRECT_URL="file:./dev.db"
NEXT_PUBLIC_SUPABASE_URL="your-supabase-url"
NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"
SUPABASE_SERVICE_ROLE_KEY="your-service-role-key"

# Gemini
GEMINI_API_KEY="your-gemini-api-key"

# Resend (Phase 6에서 사용)
RESEND_API_KEY="your-resend-api-key"
```

### 설치할 패키지
- next, react, react-dom (Next.js 기본)
- prisma, @prisma/client (ORM)
- @supabase/supabase-js, @supabase/ssr (인증 + Storage)
- ai, @ai-sdk/google (Vercel AI SDK + Gemini)
- tailwindcss, postcss, autoprefixer (스타일링)
- swr (데이터 패칭 + polling)
- pdf-lib (PDF 생성)
- zod (스키마 검증)

### shadcn/ui 설치 컴포넌트
Button, Card, Table, Dialog, Badge, Input, Textarea, Select, Toast, Tabs, Alert

### 제약사항
- TypeScript 사용 필수
- App Router만 사용 (Pages Router 금지)
- src/ 디렉토리 사용하지 않음
- CSS 파일 직접 작성 금지 (Tailwind만)
```

### 생성 산출물
- [ ] Next.js 프로젝트 (`npx create-next-app` 완료)
- [ ] shadcn/ui 초기화 + 11개 컴포넌트 설치
- [ ] `.env.local` 템플릿 파일
- [ ] 폴더 구조 생성 완료
- [ ] `npm run dev` 정상 실행 확인

### 검증 체크리스트
- [ ] `localhost:3000` 접속 시 기본 페이지 표시
- [ ] `npx prisma --version` 정상 출력
- [ ] shadcn/ui Button 컴포넌트 렌더링 확인
- [ ] TypeScript 에러 0개

---

## Phase 1 — 데이터 기반 구축 (Schema + Auth)

### 목표
Prisma 스키마 정의, DB 마이그레이션, Supabase Auth 설정, Middleware RBAC 구현

### 선행 조건
- Phase 0 완료

### 연계 SRS 요구사항
- SRS §3.8 ERD (Prisma Schema 전체)
- REQ-NF-020 (Supabase Auth)
- REQ-NF-024 (RBAC: OPERATOR / ADMIN / AUDITOR)
- REQ-NF-025 (감사 로그 5년 보존 — retention_until)

### AI 프롬프트

```markdown
## Task: Prisma 스키마 + Supabase Auth + RBAC 미들웨어

### 요청 1: Prisma 스키마
SRS v0.3 §3.8의 Prisma Schema를 `prisma/schema.prisma`에 그대로 작성해줘.

핵심 엔터티 8개:
- User (user_id, email, name_masked, role, password_hash, mfa_enabled, status)
- Batch (batch_id, batch_name, total_applicants, status, created_by)
- Applicant (applicant_id, batch_id, name_masked, phone_masked, email)
- Document (doc_id, applicant_id, doc_type, raw_file_path, ocr_extracted_json, ocr_confidence_score)
- VerificationJob (job_id, doc_id, check_layer, status, agency_api_response, discrepancy_detail, confidence_score, rpa_used, mock_used, reviewer_id, decision_reason)
- AuditTrail (trail_id, job_id(1:1), capture_file_path, timestamp_hash, viewer_log, is_immutable, retention_until)
- Notification (noti_id, applicant_id, job_id, channel, status, resubmit_link, resubmit_token)
- Report (report_id, batch_id, generated_by, pdf_path, status, total_count)

Enum 정의:
- UserRole: OPERATOR, ADMIN, AUDITOR
- UserStatus: ACTIVE, LOCKED, DEACTIVATED
- BatchStatus: OPEN, IN_PROGRESS, COMPLETED
- DocType: DEGREE, CERTIFICATE, CAREER
- CheckLayer: INPUT, OCR, AGENCY
- JobStatus: PENDING, IN_PROGRESS, PASS, FAIL, MANUAL_REVIEW, API_UNAVAILABLE, APPROVED, REJECTED

제약:
- JSON 컬럼은 모두 String(Text) 타입으로 선언 (SQLite 호환)
- 로컬에서는 SQLite, 배포 시 PostgreSQL로 전환 (datasource 주석 처리 방식)
- `retention_until`은 Prisma 레벨이 아닌 Server Action에서 `created_at + 5년` 계산

### 요청 2: Supabase Auth 설정
`lib/supabase/` 폴더에 다음 파일을 생성해줘:
- `client.ts` — 브라우저용 Supabase 클라이언트
- `server.ts` — Server Action/Route Handler용 서버 클라이언트
- `middleware.ts` — 미들웨어용 클라이언트

### 요청 3: Middleware RBAC
`middleware.ts`에 다음 로직 구현:
- Supabase Auth 세션 확인
- 미인증 시 `/login` 리다이렉트
- `/review/*` 경로 → ADMIN 이상만 접근
- `/api/reports/*` 경로 → ADMIN, AUDITOR만 접근
- 그 외 `(dashboard)` 그룹 → OPERATOR 이상 접근

### 요청 4: Seed 데이터
`prisma/seed.ts` 파일에 다음 테스트 데이터 생성:
- Admin 사용자 1명 (role: ADMIN)
- Operator 사용자 1명 (role: OPERATOR)
- Auditor 사용자 1명 (role: AUDITOR)
- 테스트 배치 1개 + 지원자 3명
```

### 생성 산출물
- [ ] `prisma/schema.prisma` — 8개 엔터티 + 6개 Enum
- [ ] `lib/supabase/client.ts`, `server.ts`, `middleware.ts`
- [ ] `middleware.ts` (루트) — RBAC 라우팅
- [ ] `prisma/seed.ts` — 테스트 데이터
- [ ] SQLite DB 파일 (`dev.db`) 마이그레이션 완료

### 검증 체크리스트
- [ ] `npx prisma migrate dev --name init` 성공
- [ ] `npx prisma studio` 에서 8개 테이블 확인
- [ ] Seed 데이터 삽입 후 User 3명 / Batch 1개 / Applicant 3명 확인
- [ ] 미인증 상태 `/dashboard` 접근 → `/login` 리다이렉트 확인
- [ ] OPERATOR 계정으로 `/review` 접근 → 403 반환 확인

---

## Phase 2 — 핵심 엔진 구현 (OCR + Triple Check)

### 목표
Gemini Vision OCR Server Action, Triple Check Loop 로직, Mock 응답 분기, 파일 업로드 UI 구현

### 선행 조건
- Phase 1 완료 (DB 스키마 + Auth 동작)

### 연계 SRS 요구사항
- REQ-FUNC-001 ~ 005 (F1: OCR 추출 엔진)
- REQ-FUNC-010 ~ 014 (F2: 공공 API/Mock 조회 + Triple Check)
- REQ-NF-001 (OCR p95 ≤ 20초)
- REQ-NF-002 (Triple Check p95 ≤ 30초)

### AI 프롬프트

```markdown
## Task: OCR 엔진 + Triple Check Loop 구현

### 요청 1: Gemini Vision OCR Service
`lib/gemini/ocr-service.ts` 파일을 생성해줘.

기능:
- Vercel AI SDK의 `generateObject()` 사용
- Google Gemini Vision 모델에 이미지(Base64) 전송
- Zod 스키마로 응답 구조 강제:
  ```typescript
  const OcrResultSchema = z.object({
    fields: z.object({
      name: z.string().describe("성명"),
      issueNumber: z.string().describe("발급번호/문서확인번호"),
      issueDate: z.string().describe("발급일자 (YYYY-MM-DD)"),
      issuerName: z.string().describe("발급기관명"),
    }),
    confidenceScore: z.number().min(0).max(100).describe("전체 OCR 신뢰도 점수"),
  });
  ```
- doc_type(DEGREE/CERTIFICATE/CAREER)별 프롬프트 분기
- 에러 핸들링: Gemini API 장애 시 `{ confidenceScore: 0, error: "..." }` 반환

### 요청 2: Agency API Client (Mock 포함)
`lib/actions/agency-api.ts` 파일 생성:

- `callGov24Api(docNum, issueDate)` — 정부24 API 실제 호출 (10초 타임아웃)
- `callHrdkApi(certNum, birthDate)` — HRDK API (현재 Mock)
- `mockAgencyResponse(docType)` — Mock 데이터 반환

분기 로직:
```
if (환경변수 GOV24_API_KEY 존재 && docType === 'DEGREE') {
  실제 API 호출 시도
  타임아웃(10초) → mockAgencyResponse() 대체 + mock_used=true
} else {
  mockAgencyResponse() + mock_used=true
}
```

Mock 응답 형식:
```json
{
  "verified": true,
  "issuerInfo": { "name": "서울대학교", "code": "MOCK-001" },
  "issueDate": "2024-02-15",
  "mockUsed": true
}
```

### 요청 3: Triple Check Loop Server Action
`lib/actions/verification.ts` 파일 생성:

핵심 함수: `uploadAndVerify(formData: FormData)`

실행 순서:
1. 파일 형식 검증 (PDF/JPG/PNG, ≤ 20MB) → 실패 시 HTTP 422
2. Supabase Storage에 원본 파일 저장 → raw_file_path 획득
3. DOCUMENT 레코드 생성 (status=PENDING)
4. Gemini Vision OCR 실행 → ocr_extracted_json, ocr_confidence_score 저장
5. 신뢰도 < 70 → MANUAL_REVIEW 저장 후 리턴
6. 신뢰도 ≥ 70 → Triple Check 시작:
   - Layer 1: 입력값(applicantInput) vs OCR값 비교
   - Layer 2: OCR값 vs 공공 API/Mock 조회값 비교
   - Layer 3: 최종 판정 (모두 일치 → PASS, 불일치 → FAIL + discrepancy_detail)
7. VERIFICATION_JOB 레코드 저장
8. (다음 Phase) Audit Trail 생성 호출

각 Layer 비교 시:
- 날짜 형식 정규화 (`2024.02.15` ↔ `2024-02-15`)
- 공백/특수문자 정규화
- 부분 일치 허용 기준 정의 (예: 기관명 "서울대학교" vs "서울대")

### 요청 4: 서류 업로드 UI
`app/(dashboard)/verifications/new/page.tsx` 생성:
- shadcn/ui Card 레이아웃
- 파일 드래그앤드롭 + 클릭 업로드
- 서류 유형 선택 (학위증명서/자격증/경력증명서) — Select 컴포넌트
- 지원자 정보 입력 (성명, 발급번호, 발급일자 — 비교용)
- 제출 버튼 → Server Action 호출
- 로딩 상태 표시 (OCR 처리 중 안내 — 최대 20초 소요)
- 결과 표시: 성공 시 검증 결과 요약 카드, 실패 시 에러 메시지
```

### 생성 산출물
- [ ] `lib/gemini/ocr-service.ts` — Gemini Vision OCR 래핑
- [ ] `lib/actions/agency-api.ts` — 공공 API + Mock 클라이언트
- [ ] `lib/actions/verification.ts` — `uploadAndVerify()` Server Action
- [ ] `lib/utils/normalize.ts` — 날짜/문자열 정규화 유틸
- [ ] `app/(dashboard)/verifications/new/page.tsx` — 업로드 UI
- [ ] Zod 스키마 정의 파일

### 검증 체크리스트
- [ ] 실제 학위증명서 JPG 업로드 → JSON 필드 4개 추출 확인
- [ ] OCR 신뢰도 < 70 시뮬레이션 → MANUAL_REVIEW 저장 확인
- [ ] 의도적 불일치 데이터 투입 → FAIL + discrepancy_detail 확인
- [ ] 모든 일치 데이터 → PASS 확인
- [ ] Mock 사용 건 → `mock_used=true` DB 저장 확인
- [ ] 20MB 초과 파일 → HTTP 422 반환 확인
- [ ] .bmp 파일 → HTTP 422 반환 확인

---

## Phase 3 — 감사 체계 구현 (Audit Trail + 불변성)

### 목표
검증 완료 시 Audit Trail 자동 생성, SHA-256 해시, 논리적 불변성, 열람 로그 기록

### 선행 조건
- Phase 2 완료 (Triple Check Loop 동작)

### 연계 SRS 요구사항
- REQ-FUNC-020 ~ 023 (F3: Audit Trail)
- REQ-NF-025 (감사 로그 5년 보존)
- REQ-NF-026 (논리적 불변성)
- ADR-006 (WORM 대신 논리적 보장)

### AI 프롬프트

```markdown
## Task: Audit Trail 서비스 구현

### 요청 1: AuditTrailService
`lib/actions/audit-trail.ts` 파일 생성:

#### createTrail(jobId, filePath)
- VERIFICATION_JOB 완료 후 자동 호출
- capture_file_path: 원본 서류 Supabase Storage 경로 (병렬 캡처 Mock 대체)
- timestamp_hash: `SHA-256(filePath + captureTimestamp.toISOString())` 생성
  - Node.js crypto 모듈 사용
- capture_timestamp: `new Date()` (± 1초 정밀도)
- is_immutable: `true` (항상)
- retention_until: `new Date(Date.now() + 5 * 365 * 24 * 60 * 60 * 1000)` (5년)
- viewer_log: `"[]"` (빈 JSON 배열 문자열)

#### logAccess(trailId, userId)
- 기존 viewer_log JSON 파싱 → `{ userId, timestamp }` 추가 → 다시 문자열로 저장
- Prisma `update()` 사용

#### blockModification()
- Audit Trail의 DELETE / PUT / PATCH Route Handler를 아예 만들지 않음
- 만약 요청이 들어오면 HTTP 403 + `{ error: "Audit Trail은 수정/삭제할 수 없습니다." }` 반환
- 별도 `app/api/audit-trails/[trailId]/route.ts` 에서 GET만 허용

### 요청 2: Phase 2의 verification.ts 수정
`uploadAndVerify()` 함수 끝에 `createTrail()` 호출 추가:
```typescript
// Triple Check 완료 후
const trail = await createTrail(job.jobId, document.rawFilePath);
```

### 요청 3: Audit Trail 열람 API
`app/api/audit-trails/[trailId]/route.ts`:
- GET: trail 데이터 반환 + 자동으로 `logAccess()` 호출
- DELETE/PUT/PATCH: HTTP 403 반환
- RBAC: ADMIN, AUDITOR만 접근 가능
```

### 생성 산출물
- [ ] `lib/actions/audit-trail.ts` — createTrail, logAccess
- [ ] `app/api/audit-trails/[trailId]/route.ts` — GET(열람) + 403(수정 차단)
- [ ] Phase 2 `verification.ts` 수정 (createTrail 호출 추가)
- [ ] SHA-256 해시 생성 유틸 (`lib/utils/hash.ts`)

### 검증 체크리스트
- [ ] 검증 완료 후 AUDIT_TRAIL 레코드 자동 생성 확인
- [ ] timestamp_hash가 SHA-256 형식 (64자 hex) 확인
- [ ] is_immutable = true 확인
- [ ] retention_until = created_at + 5년 확인
- [ ] GET `/api/audit-trails/{id}` → viewer_log에 접근 기록 추가 확인
- [ ] DELETE `/api/audit-trails/{id}` → HTTP 403 확인
- [ ] PUT `/api/audit-trails/{id}` → HTTP 403 확인

---

## Phase 4 — 사용자 인터페이스 (Dashboard + 승인/반려)

### 목표
검증 결과 대시보드, 불일치 항목 하이라이트 상세 화면, Human-in-the-loop 승인/반려 UI, MANUAL_REVIEW 큐

### 선행 조건
- Phase 3 완료 (Audit Trail 동작)

### 연계 SRS 요구사항
- REQ-FUNC-014 (불일치 항목 하이라이트, 2초 이내)
- REQ-FUNC-030 ~ 033 (F4: Human-in-the-loop 승인 UI)
- REQ-FUNC-060 ~ 061 (F7: 대시보드 통계)
- REQ-NF-004 (대시보드 p95 ≤ 3초)
- REQ-NF-005 (승인/반려 p95 ≤ 2초)
- ADR-007 (SWR 5초 polling)

### AI 프롬프트

```markdown
## Task: 대시보드 + Human-in-the-loop UI 구현

### 요청 1: 메인 대시보드
`app/(dashboard)/dashboard/page.tsx` 생성:

화면 구성 (shadcn/ui 사용):
- 상단 통계 카드 4개: 총 검증 건수 / PASS 건수 / FAIL 건수 / MANUAL_REVIEW 건수
- 추가 카드 1개: Mock 사용 건수 (Badge로 표시)
- 하단 최근 검증 목록 테이블 (Table 컴포넌트):
  - 컬럼: 지원자(마스킹), 서류유형, 상태(Badge 색상 분기), 신뢰도, Mock여부, 검증일시
  - 행 클릭 → 상세 화면 이동

데이터 패칭:
- `useSWR('/api/dashboard/stats', fetcher, { refreshInterval: 5000 })` — 5초 polling
- Route Handler `app/api/dashboard/stats/route.ts` → Prisma 집계 쿼리

상태별 Badge 색상:
- PASS: green
- FAIL: red (destructive)
- MANUAL_REVIEW: yellow (warning)
- APPROVED: blue
- REJECTED: gray
- API_UNAVAILABLE: orange

### 요청 2: 검증 결과 상세 화면
`app/(dashboard)/verifications/[jobId]/page.tsx` 생성:

화면 구성:
- 좌측: 원본 서류 이미지 표시 (Supabase Storage URL)
- 우측: 검증 결과 카드
  - OCR 추출값 표시 (필드별)
  - 기관 DB 반환값 표시 (필드별)
  - **불일치 항목 하이라이트**: 값이 다른 필드에 빨간색 배경 + 텍스트 비교
  - confidence_score 게이지 표시
  - mock_used 표시 (Mock 사용 건 별도 Badge)

불일치 하이라이트 구현:
```tsx
// 각 필드를 순회하며 OCR값과 API값 비교
{fields.map(field => (
  <div className={ocrValue !== apiValue ? "bg-red-50 border-red-500" : "bg-green-50"}>
    <span>OCR: {ocrValue}</span>
    <span>API: {apiValue}</span>
  </div>
))}
```

### 요청 3: 승인/반려 Server Actions
`lib/actions/review.ts` 생성:

- `approveJob(jobId: string)`:
  - status → APPROVED
  - reviewer_id → 현재 로그인 사용자 ID
  - reviewed_at → now()
  - AUDIT_TRAIL.viewer_log에 승인 이력 추가
  - RBAC 확인: ADMIN 이상만 가능

- `rejectJob(jobId: string, reason: string)`:
  - reason 비어있으면 에러 반환 (사유 필수)
  - status → REJECTED
  - decision_reason → reason
  - AUDIT_TRAIL.viewer_log에 반려 이력 추가

### 요청 4: 승인/반려 UI
검증 결과 상세 화면 하단에:
- '승인' 버튼 (green) → 확인 Dialog → approveJob() 호출
- '반려' 버튼 (red) → 사유 입력 Textarea 포함 Dialog → rejectJob() 호출
- 사유가 비어있으면 '반려' 제출 버튼 비활성

### 요청 5: MANUAL_REVIEW 큐 화면
`app/(dashboard)/review/page.tsx` 생성:
- MANUAL_REVIEW 상태 건만 필터링 표시
- 건별 상세 확인 + 수동 처리 + 상태 변경 가능
- ADMIN 이상만 접근 (Middleware에서 이미 처리)
```

### 생성 산출물
- [ ] `app/(dashboard)/dashboard/page.tsx` — 메인 대시보드
- [ ] `app/api/dashboard/stats/route.ts` — 통계 API
- [ ] `app/(dashboard)/verifications/[jobId]/page.tsx` — 상세 화면
- [ ] `lib/actions/review.ts` — approveJob, rejectJob
- [ ] `app/(dashboard)/review/page.tsx` — MANUAL_REVIEW 큐

### 검증 체크리스트
- [ ] 대시보드 5개 통계 카드 정상 표시
- [ ] SWR 5초 polling 동작 확인 (새 검증 후 자동 갱신)
- [ ] FAIL 건 상세 화면 → 불일치 항목 빨간색 하이라이트 확인
- [ ] '승인' 클릭 → APPROVED 상태 변경 + viewer_log 기록 확인
- [ ] '반려' 사유 미입력 → 제출 버튼 비활성 확인
- [ ] '반려' 사유 입력 → REJECTED + decision_reason 저장 확인
- [ ] OPERATOR 계정 `/review` 접근 → 차단 확인

---

## Phase 5 — 출력물 생성 (PDF 리포트)

### 목표
pdf-lib 기반 감사 PDF 리포트 동기 생성, 브라우저 다운로드

### 선행 조건
- Phase 4 완료 (승인/반려 동작)

### 연계 SRS 요구사항
- REQ-FUNC-040 ~ 042 (F5: PDF 리포트)
- REQ-NF-003 (p95 ≤ 10초, 100건 이하)

### AI 프롬프트

```markdown
## Task: PDF 감사 리포트 생성

### 요청 1: ReportService
`lib/actions/report.ts` 생성:

#### generateReport(batchId: string)
1. Prisma로 해당 배치의 전체 VERIFICATION_JOB + AUDIT_TRAIL 조회
2. pdf-lib으로 PDF 생성:
   - 1페이지: 표지
     - 제목: "HR AI 서류 진위확인 감사 리포트"
     - 회차명 (batch.batchName)
     - 생성일시
     - 총 건수 / PASS / FAIL / MANUAL_REVIEW 요약
   - 2페이지~: 건별 결과 테이블
     - 컬럼: #, 지원자(마스킹), 서류유형, 판정결과, 신뢰도, Mock여부, 검증일시
     - **mock_used=true 건은 결과 옆에 "(Mock)" 표기**
     - FAIL 건: 불일치 항목 요약 하단 표시
   - 마지막 페이지: 면책 조항
     - "본 리포트는 AI 보조 검증 결과이며, 최종 판단은 담당자 승인에 기반합니다."
     - "Mock 데이터 사용 건은 실제 기관 검증이 미완료된 상태입니다."

3. PDF Buffer 반환

주의: 한글 폰트 처리
- pdf-lib에서 한글 표시를 위해 커스텀 폰트 임베드 필요
- `public/fonts/NotoSansKR-Regular.ttf` 파일을 프로젝트에 포함
- `pdfDoc.embedFont(fontBytes)` 사용

### 요청 2: PDF 다운로드 Route Handler
`app/api/reports/[batchId]/pdf/route.ts`:
- GET 요청 → generateReport(batchId) → PDF Buffer 반환
- Content-Type: application/pdf
- Content-Disposition: attachment; filename="audit-report-{batchId}.pdf"
- RBAC: ADMIN, AUDITOR만 접근

### 요청 3: 다운로드 UI
대시보드 또는 배치 상세 화면에:
- '감사 리포트 다운로드' 버튼
- 클릭 → `/api/reports/{batchId}/pdf` 로 브라우저 다운로드 트리거
- 로딩 상태 표시 (최대 10초)
```

### 생성 산출물
- [ ] `lib/actions/report.ts` — generateReport
- [ ] `app/api/reports/[batchId]/pdf/route.ts` — PDF 다운로드 API
- [ ] `public/fonts/NotoSansKR-Regular.ttf` — 한글 폰트 파일
- [ ] 다운로드 버튼 UI 컴포넌트

### 검증 체크리스트
- [ ] 10건 배치 기준 PDF 생성 → 10초 이내 완료
- [ ] PDF 내 회차명, 검증일시, 건별 결과 포함 확인
- [ ] mock_used=true 건에 "(Mock)" 표기 확인
- [ ] 한글 깨짐 없이 정상 표시 확인
- [ ] FAIL 건 불일치 항목 요약 포함 확인
- [ ] OPERATOR 계정 → PDF 다운로드 접근 차단 확인

---

## Phase 6 — 알림 체계 (이메일 알림)

### 목표
FAIL 확정 시 Resend API 이메일 발송, 실패 시 대시보드 알림 대체

### 선행 조건
- Phase 5 완료
- Resend 계정 생성 + API 키 발급

### 연계 SRS 요구사항
- REQ-FUNC-050 ~ 051 (F6: 이메일 알림)

### AI 프롬프트

```markdown
## Task: Resend 이메일 알림 구현

### 요청 1: NotificationService
`lib/actions/notification.ts` 생성:

#### sendFailNotification(jobId: string)
1. VERIFICATION_JOB + APPLICANT + DOCUMENT 조인 조회
2. 이메일 내용 구성:
   - 제목: "[HR AI 검증] 서류 불일치 발견 — {지원자명(마스킹)}"
   - 본문:
     - 지원자: 홍○○
     - 서류 유형: 학위증명서
     - 불일치 항목: 발급번호 (OCR: XXX vs 기관DB: YYY)
     - 대시보드 링크: `{BASE_URL}/verifications/{jobId}`
3. Resend API 호출:
   ```typescript
   import { Resend } from 'resend';
   const resend = new Resend(process.env.RESEND_API_KEY);
   await resend.emails.send({
     from: 'noreply@yourdomain.com',
     to: operatorEmail,
     subject: title,
     html: emailHtml,
   });
   ```
4. 발송 성공 → NOTIFICATION 레코드 (channel=EMAIL, status=SENT)
5. 발송 실패 → NOTIFICATION (status=FAILED) + 대시보드 알림 텍스트 저장

### 요청 2: Phase 2 verification.ts 수정
Triple Check 결과 FAIL 시 `sendFailNotification()` 호출 추가:
```typescript
if (finalStatus === 'FAIL') {
  await sendFailNotification(job.jobId);
}
```
```

### 생성 산출물
- [ ] `lib/actions/notification.ts` — sendFailNotification
- [ ] 이메일 HTML 템플릿 (인라인 스타일)
- [ ] Phase 2 `verification.ts` 수정 (알림 호출 추가)

### 검증 체크리스트
- [ ] FAIL 건 발생 → 이메일 발송 확인 (Resend Dashboard)
- [ ] 이메일 내용: 지원자명(마스킹), 서류유형, 불일치 항목, 링크 4개 포함
- [ ] Resend API 키 없을 때 → 대시보드 알림 텍스트 표시 확인
- [ ] NOTIFICATION 레코드 생성 확인

---

## Phase 7 — 통합 테스트 + 배포

### 목표
E2E 흐름 검증, Supabase PostgreSQL 전환, Vercel 배포

### 선행 조건
- Phase 0~6 모두 완료

### 연계 SRS 요구사항
- Validation Plan Exp-1 ~ Exp-5 전체
- C-TEC-003 (SQLite → PostgreSQL 전환)
- C-TEC-007 (Vercel 배포)

### AI 프롬프트

```markdown
## Task: 통합 테스트 + Vercel 배포

### 요청 1: E2E 테스트 시나리오 실행 가이드
아래 5개 실험을 순서대로 실행하기 위한 단계별 가이드를 작성해줘:

| 실험 | 방법 | 성공 기준 |
|---|---|---|
| Exp-1 | 실제 학위증명서 10건 OCR | 핵심 필드 정확도 ≥ 80% |
| Exp-2 | 의도적 불일치 5건 투입 | 5건 모두 FAIL + discrepancy_detail 정확 |
| Exp-3 | DELETE API 직접 호출 | HTTP 403 + 레코드 유지 |
| Exp-4 | 10건 배치 PDF 생성 | 필수 항목 누락 0건 |
| Exp-5 | 업로드→검증→승인→PDF 전체 흐름 | 오류 없이 완료 |

### 요청 2: Prisma 데이터소스 전환
`prisma/schema.prisma`의 datasource를 PostgreSQL로 전환:
```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}
```

마이그레이션 명령:
```bash
npx prisma migrate deploy
npx prisma generate
```

### 요청 3: Vercel 배포 설정
1. GitHub 리포지토리 연결
2. Vercel 환경변수 설정:
   - DATABASE_URL (Supabase PostgreSQL connection string)
   - DIRECT_URL (Supabase direct connection)
   - NEXT_PUBLIC_SUPABASE_URL
   - NEXT_PUBLIC_SUPABASE_ANON_KEY
   - SUPABASE_SERVICE_ROLE_KEY
   - GEMINI_API_KEY
   - RESEND_API_KEY
3. `git push origin main` → 자동 배포
4. 배포 후 E2E Exp-5 재실행

### 요청 4: 배포 후 점검 리스트
- [ ] Vercel 프로덕션 URL 접속 가능
- [ ] 로그인 → 대시보드 접근 가능
- [ ] 파일 업로드 → OCR → Triple Check 정상 동작
- [ ] PDF 다운로드 정상 동작
- [ ] Supabase Dashboard에서 데이터 확인
```

### 생성 산출물
- [ ] E2E 테스트 실행 가이드 문서
- [ ] Prisma 스키마 PostgreSQL 전환 완료
- [ ] Vercel 배포 완료 (프로덕션 URL 확보)
- [ ] 배포 후 점검 리스트 통과

### 검증 체크리스트
- [ ] Exp-1 ~ Exp-5 전체 통과
- [ ] Vercel 프로덕션 환경 E2E 흐름 정상 동작
- [ ] Supabase PostgreSQL 데이터 정상 저장
- [ ] 월 운영 비용 $0~$5 범위 내 확인

---
---

## 부록 A — Phase별 의존 관계 다이어그램

```
Phase 0 (Scaffolding)
   │
   ▼
Phase 1 (Schema + Auth)
   │
   ▼
Phase 2 (OCR + Triple Check)  ◄── 핵심 엔진, 가장 높은 난이도
   │
   ▼
Phase 3 (Audit Trail)
   │
   ▼
Phase 4 (Dashboard + 승인/반려 UI)
   │
   ▼
Phase 5 (PDF 리포트)
   │
   ▼
Phase 6 (이메일 알림)  ◄── Should 우선순위, 건너뛰기 가능
   │
   ▼
Phase 7 (통합 테스트 + 배포)
```

## 부록 B — Phase ↔ SRS REQ-FUNC 매핑

| Phase | REQ-FUNC ID | 기능 |
|---|---|---|
| 0 | — | 프로젝트 초기화 |
| 1 | — (인프라) | Schema + Auth + RBAC |
| 2 | 001~005, 010~014 | OCR + Triple Check |
| 3 | 020~023 | Audit Trail |
| 4 | 014, 030~033, 060~061 | Dashboard + 승인/반려 |
| 5 | 040~042 | PDF 리포트 |
| 6 | 050~051 | 이메일 알림 |
| 7 | Exp-1~5 | 통합 검증 |

## 부록 C — 각 Phase 예상 소요 시간

| Phase | 예상 소요 | 난이도 | 비고 |
|---|---|---|---|
| 0 | 0.5~1일 | ⭐ | 셋업 자동화 가능 |
| 1 | 1~1.5일 | ⭐⭐ | Prisma 스키마가 SRS에 포함되어 있어 빠름 |
| 2 | 3~4일 | ⭐⭐⭐ | **가장 핵심**, Gemini API + 비교 로직 |
| 3 | 1일 | ⭐⭐ | Phase 2에 연결만 하면 됨 |
| 4 | 2일 | ⭐⭐ | UI 컴포넌트 조합 위주 |
| 5 | 1~2일 | ⭐⭐⭐ | pdf-lib 한글 폰트 처리가 난관 |
| 6 | 0.5일 | ⭐ | Resend SDK가 매우 단순 |
| 7 | 1일 | ⭐⭐ | 수동 테스트 + 배포 |
| **합계** | **10~13일** | | 바이브코딩 기준 약 2~3주 |

## 부록 D — AI 프롬프트 사용 시 주의사항

```
✅ 프롬프트 앞에 항상 System Context 삽입

✅ Phase 순서 반드시 준수 (의존 관계)

✅ 각 Phase 완료 후 검증 체크리스트 전체 통과 확인 후 다음 Phase 진행

✅ AI가 C-TEC 제약 외 도구를 제안하면 즉시 거절하고 재지시

✅ 에러 발생 시 에러 메시지 전체를 AI에게 복사하여 디버깅 요청

❌ 여러 Phase를 동시에 진행하지 않음 (의존성 꼬임 방지)

❌ AI가 생성한 코드를 검증 없이 다음 Phase로 넘기지 않음

❌ Phase 2의 OCR/Triple Check를 건너뛰고 UI부터 만들지 않음
```

---

*— End of SRS Revision Recipe v0.3 —*

**작성일:** 2026-04-17  
**기준 문서:** SRS-HR-AI-Verification-v0.3.md  
**용도:** AI 에이전트 단계별 오케스트레이팅 프롬프트  
**대상:** 입문자 바이브코딩 개발자 + AI 코드 생성 에이전트 (Claude, Cursor, Windsurf)
