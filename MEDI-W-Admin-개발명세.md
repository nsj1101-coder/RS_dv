# MEDI-W Admin — 개발 명세서

> **문서 버전:** v1.0  
> **작성일:** 2026-04-17  
> **현재 단계:** 정적 HTML 시안 완성 → 실제 앱 개발 전환

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [시안 현황](#2-시안-현황)
3. [전체 개발 프로세스](#3-전체-개발-프로세스)
4. [비즈니스 플로우](#4-비즈니스-플로우)
5. [화면별 기능 명세](#5-화면별-기능-명세)
6. [API 명세](#6-api-명세)
7. [데이터 모델](#7-데이터-모델)
8. [개발 로드맵](#8-개발-로드맵)
9. [기술 스택](#9-기술-스택)

---

## 1. 프로젝트 개요

**MEDI-W**는 해외 환자를 국내 병원·의료 기관과 연결하는 글로벌 의료 허브 플랫폼이다.  
이 문서는 MEDI-W **어드민 시스템**(슈퍼 어드민 전용)의 개발 명세를 정리한다.

### 이해관계자

| 역할 | 시스템 | 주요 업무 |
|------|--------|----------|
| **슈퍼 어드민** | MEDI-W Admin ← **이 문서** | 병원 심사/승인, 에이전시 관리, 통계, 시스템 설정 |
| 병원 담당자 | 병원 포털 (별도 개발) | 병원 정보 관리, 상담 수신, 일정 관리 |
| 에이전시 파트너 | 에이전시 포털 (별도 개발) | 환자 유치, 상담 신청, 수수료 정산 |
| 해외 환자 | 환자 앱/웹 (별도 개발) | 병원 검색, 상담 신청, 예약 확인 |

> ⚠️ 이 어드민은 **슈퍼 어드민 전용** 시스템이다. 병원·에이전시·환자 포털은 별도 시스템으로 개발한다.

---

## 2. 시안 현황

### 완성된 시안 파일 구조

```
RS_dv/
├── pages/          # v1 — 23개 화면 (슬라이드 패널 방식)
├── pages-v2/       # v2 — 24개 화면 (별도 페이지 방식)
├── assets/
│   ├── css/common.css   # 공통 디자인 시스템 (330줄)
│   └── js/common.js     # 공통 인터랙션 (214줄)
└── medi-w-admin-preview.html  # 풀스크린 프리뷰
```

### v1 vs v2 차이

| 항목 | v1 (pages/) | v2 (pages-v2/) |
|------|-------------|----------------|
| 환자 등록 | 슬라이드 패널(모달) | 별도 페이지 (`patient-register.html`) |
| 파일 수 | 23개 | 24개 |
| 나머지 | 동일 | 동일 |

**권장:** v2 채택 (더 나은 폼 공간, 브라우저 백버튼 지원)

### 화면 목록 (23개)

| 그룹 | 파일 | 화면명 |
|------|------|--------|
| 인증 | `login.html` | 로그인 |
| 인증 | `session-expired.html` | 세션 만료 |
| 대시보드 | `dashboard.html` | 메인 대시보드 |
| 대시보드 | `notification.html` | 알림 센터 |
| 병원 | `hospital-search.html` | 병원 검색 |
| 병원 | `hospital-list.html` | 병원 관리 |
| 병원 | `hospital-register.html` | 병원 등록 |
| 환자·상담 | `patient.html` | 환자 관리 |
| 환자·상담 | `patient-register.html` | 환자 등록 (v2 전용) |
| 환자·상담 | `consultation.html` | 상담/예약 관리 |
| 파트너 | `agency.html` | 에이전시 관리 |
| 통계 | `statistics.html` | 통계 허브 |
| 통계 | `statistics-operations.html` | 운영 통계 |
| 통계 | `statistics-patient.html` | 환자 분석 |
| 통계 | `statistics-revenue.html` | 수익 분석 |
| 내 정보 | `my-info.html` | 관리자 정보 |
| 설정 | `settings.html` | 설정 허브 |
| 설정 | `settings-general.html` | 일반 설정 |
| 설정 | `settings-notification.html` | 알림 설정 |
| 설정 | `settings-permission.html` | 권한 설정 |
| 설정 | `settings-data.html` | 데이터 관리 |
| 오류 | `404.html` | 404 Not Found |
| 오류 | `403.html` | 403 Forbidden |

---

## 3. 전체 개발 프로세스

### 시안 → 실제 앱 전환 단계

```
[1] 기반 세팅
  └─ 레포 생성 (Next.js 16 + TypeScript + Tailwind 4)
  └─ DB 스키마 설계 (Prisma 7 + PostgreSQL)
  └─ 공통 레이아웃 컴포넌트화 (Sidebar, TopBar, Layout)
  └─ common.css → Tailwind 클래스 변환 또는 CSS Module 유지

[2] 인증 구현
  └─ JWT 발급 (Access Token 15분, Refresh Token 7일)
  └─ HttpOnly Cookie 저장
  └─ 미들웨어 인증 가드
  └─ 로그인/로그아웃 페이지 연동

[3] 도메인별 CRUD 개발 (Phase 1~2 순서)
  └─ 각 화면의 정적 HTML → React 컴포넌트
  └─ 하드코딩 샘플 데이터 → API fetch 교체
  └─ 폼 → react-hook-form + zod 유효성 검사

[4] 외부 서비스 연동
  └─ 카카오맵 SDK (병원 검색 지도)
  └─ AWS S3 presigned URL (파일 업로드)
  └─ SSE (실시간 알림)
  └─ 이메일 발송 (SendGrid / AWS SES)

[5] 통계 및 고도화
  └─ Chart.js 또는 Recharts 동적 차트
  └─ 집계 쿼리 최적화 (Prisma aggregate)
  └─ CSV 내보내기

[6] QA 및 배포
  └─ 반응형 완성 (1024px 이하)
  └─ 접근성 (ARIA, 키보드 내비게이션)
  └─ Vercel 또는 AWS 배포
```

### 시안 변환 시 주요 작업

| 항목 | 현재 (시안) | 실제 앱 |
|------|------------|---------|
| 데이터 | 하드코딩 HTML | API 응답 |
| 차트 | CSS conic-gradient / 정적 SVG | Chart.js / Recharts |
| 지도 | 이미지 플레이스홀더 | 카카오맵 SDK |
| 파일 업로드 | `input[type=file]` UI만 | S3 presigned URL |
| 알림 | 정적 리스트 | SSE 실시간 스트림 |
| 칸반 드래그 | CSS 전환만 | `@dnd-kit` |
| 인증 | 없음 | JWT + HttpOnly Cookie |
| 드래그 순서 | 없음 | Patch API 연동 |

---

## 4. 비즈니스 플로우

### 병원 온보딩

```
병원 담당자 → 병원 등록 신청
           → [어드민] 서류 심사
           → [어드민] 승인 / 반려
           → 승인 시: 병원 ACTIVE 상태로 플랫폼 노출
           → 반려 시: 사유 이메일 발송
```

### 환자 상담 프로세스

```
환자/에이전시 → 상담 신청 (PENDING)
             → [어드민] 병원 배정 (IN_PROGRESS)
             → 병원 예약 확정 (CONFIRMED)
             → 진료 완료 (COMPLETED)
             → 수수료 정산 → 에이전시
```

### 상담 칸반 상태 정의

| 상태 키 | 표시명 | 설명 | 다음 액션 |
|---------|--------|------|----------|
| `PENDING` | 신청 접수 | 상담 신청 완료 | 병원 배정 또는 반려 |
| `IN_PROGRESS` | 진행 중 | 병원 연결, 상담 진행 | 예약 확정 또는 취소 |
| `CONFIRMED` | 예약 확정 | 날짜/시간 확정 | 진료 완료 처리 |
| `COMPLETED` | 완료 | 진료 완료 | 수수료 정산 |

---

## 5. 화면별 기능 명세

### 대시보드 (`dashboard.html`)

| UI 요소 | 기능 | API | 우선순위 |
|---------|------|-----|---------|
| 통계 카드 4종 | 전체 병원 수, 금월 신규 환자, 진행 중 상담, 총 매출 | `GET /stats/summary` | P1 |
| 도넛 차트 | 상태별 상담 비율 | `GET /stats/consultation-status` | P2 |
| 최근 등록 병원 | 최신 5개 | `GET /hospitals?limit=5&orderBy=createdAt:desc` | P1 |
| 최근 알림 | 미읽음 상위 5개 | `GET /notifications?unread=true&limit=5` | P2 |

### 병원 관리 (`hospital-list.html`, `hospital-register.html`, `hospital-search.html`)

| UI 요소 | 기능 | API | 우선순위 |
|---------|------|-----|---------|
| 병원 목록 테이블 | 페이지네이션, 정렬, 상태 필터 | `GET /hospitals` | P1 |
| 검색 | 병원명, 지역, 진료과 | `GET /hospitals?q=&region=&specialty=` | P1 |
| 상태 변경 | active / inactive / pending 토글 | `PATCH /hospitals/:id/status` | P1 |
| 병원 등록 폼 | 멀티 스텝, 이미지 업로드 | `POST /hospitals` (multipart) | P1 |
| 지도 검색 | 카카오맵 연동, 위치 선택 | 카카오맵 API | P2 |
| 드래그 순서 변경 | 노출 순서 일괄 저장 | `PATCH /hospitals/order` | P3 |

### 환자 관리 (`patient.html`, `patient-register.html`)

| UI 요소 | 기능 | API | 우선순위 |
|---------|------|-----|---------|
| 환자 목록 (카드/리스트/칸반) | 뷰 전환, 페이지네이션 | `GET /patients` | P1 |
| 검색/필터 | 이름, 국가, 상태, 담당 병원 | `GET /patients?q=&country=&status=` | P1 |
| 슬라이드 패널 | 상세 조회, 인라인 편집 | `GET + PATCH /patients/:id` | P1 |
| 환자 등록 폼 | 기본 정보, 의료 이력, 연락처 | `POST /patients` | P1 |

### 상담/예약 관리 (`consultation.html`)

| UI 요소 | 기능 | API | 우선순위 |
|---------|------|-----|---------|
| 칸반 보드 4컬럼 | 드래그&드롭 상태 이동 | `PATCH /consultations/:id/status` | P1 |
| 통계 요약 행 | 컬럼별 건수 | `GET /consultations/counts` | P1 |
| 병원 배정 | 상담에 병원 연결 | `PATCH /consultations/:id/assign` | P1 |

### 통계 (`statistics-*.html`)

| UI 요소 | 기능 | API | 우선순위 |
|---------|------|-----|---------|
| 기간 필터 | 오늘/7일/30일/3개월/직접입력 | 쿼리 파라미터 `from`, `to` | P2 |
| 운영 통계 | 신규 병원/환자 추이, 상담 전환율 | `GET /stats/operations` | P2 |
| 환자 분석 | 국가별, 연령별, 성별 분포 | `GET /stats/patients` | P2 |
| 수익 분석 | 매출 추이, 수수료 분포 | `GET /stats/revenue` | P2 |
| CSV 내보내기 | 조회 데이터 다운로드 | `GET /stats/export` | P3 |

---

## 6. API 명세

**Base URL:** `https://api.medi-w.com/v1`  
**인증:** 모든 API에 `Authorization: Bearer <access_token>` 필요 (로그인 제외)

### 6.1 인증 (Auth)

```
POST   /auth/login         # 로그인 → Access + Refresh Token
POST   /auth/logout        # Refresh Token 무효화
POST   /auth/refresh       # Access Token 갱신
GET    /auth/me            # 현재 어드민 정보
PATCH  /auth/password      # 비밀번호 변경
```

**로그인 요청/응답 예시:**
```json
// POST /auth/login
{ "email": "admin@medi-w.com", "password": "..." }

// Response
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ...",
  "admin": { "id": "...", "name": "홍길동", "role": "SUPER_ADMIN" }
}
```

### 6.2 병원 (Hospitals)

```
GET    /hospitals                    # 목록 (q, region, specialty, status, page, limit)
GET    /hospitals/:id                # 상세
POST   /hospitals                    # 등록 (multipart/form-data)
PUT    /hospitals/:id                # 전체 수정
PATCH  /hospitals/:id/status         # 상태 변경
PATCH  /hospitals/order              # 노출 순서 일괄 업데이트
DELETE /hospitals/:id                # 삭제 (소프트)
GET    /hospitals/search             # 지도 검색 (lat, lng, radius, keyword)
```

### 6.3 환자 (Patients)

```
GET    /patients                     # 목록 (q, country, status, hospitalId, from, to)
GET    /patients/:id                 # 상세 + 상담 이력
POST   /patients                     # 등록
PATCH  /patients/:id                 # 수정
DELETE /patients/:id                 # 삭제 (소프트)
GET    /patients/:id/consultations   # 상담 이력
```

### 6.4 상담/예약 (Consultations)

```
GET    /consultations                # 목록 (status, hospitalId, patientId, from, to)
GET    /consultations/counts         # 칸반 컬럼별 건수
GET    /consultations/:id            # 상세
POST   /consultations                # 등록
PATCH  /consultations/:id/status     # 상태 변경 (칸반 드래그)
PATCH  /consultations/:id/assign     # 병원 배정 { hospitalId }
```

### 6.5 에이전시 (Agencies)

```
GET    /agencies                     # 목록 (country, status, sort)
GET    /agencies/:id                 # 상세 + 실적
POST   /agencies                     # 등록
PATCH  /agencies/:id                 # 수정
PATCH  /agencies/:id/status          # 상태 변경
GET    /agencies/:id/settlements     # 수수료 정산 내역
```

### 6.6 통계 (Statistics)

```
GET    /stats/summary                # 대시보드 요약
GET    /stats/consultation-status    # 상담 상태별 비율
GET    /stats/operations             # 운영 통계 (from, to)
GET    /stats/patients               # 환자 분석 (from, to)
GET    /stats/revenue                # 수익 분석 (from, to)
GET    /stats/export                 # CSV/Excel 다운로드 (type, from, to)
```

### 6.7 알림 (Notifications)

```
GET    /notifications                # 목록 (type, unread, page)
PATCH  /notifications/:id/read      # 읽음 처리
PATCH  /notifications/read-all      # 전체 읽음
GET    /notifications/stream        # SSE 실시간 스트림
```

### 6.8 설정 (Settings)

```
GET    /settings                     # 전체 설정 조회
PATCH  /settings/general             # 일반 설정 저장
PATCH  /settings/notifications       # 알림 채널 설정
GET    /settings/permissions         # 역할별 권한 조회
PATCH  /settings/permissions         # 권한 저장
POST   /settings/backup              # 백업 실행
GET    /settings/audit-logs          # 감사 로그 (page, limit)
```

---

## 7. 데이터 모델

### Admin

```prisma
model Admin {
  id           String   @id @default(uuid())
  email        String   @unique
  passwordHash String
  name         String
  role         AdminRole @default(ADMIN)
  avatarUrl    String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  notifications Notification[]
}

enum AdminRole {
  SUPER_ADMIN
  ADMIN
}
```

### Hospital

```prisma
model Hospital {
  id           String         @id @default(uuid())
  name         String
  nameEn       String?
  region       String
  address      String?
  lat          Float?
  lng          Float?
  specialties  String[]
  phone        String?
  email        String?
  imageUrls    String[]
  status       HospitalStatus @default(PENDING)
  displayOrder Int            @default(0)
  deletedAt    DateTime?
  createdAt    DateTime       @default(now())
  updatedAt    DateTime       @updatedAt

  consultations Consultation[]
}

enum HospitalStatus {
  ACTIVE
  INACTIVE
  PENDING
}
```

### Patient

```prisma
model Patient {
  id          String   @id @default(uuid())
  name        String
  nameEn      String?
  nationality String
  passportNo  String?
  gender      Gender?
  birthDate   DateTime?
  phone       String
  email       String?
  memo        String?
  agencyId    String?
  deletedAt   DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  agency        Agency?        @relation(fields: [agencyId], references: [id])
  consultations Consultation[]
}

enum Gender {
  M
  F
  OTHER
}
```

### Consultation

```prisma
model Consultation {
  id             String             @id @default(uuid())
  patientId      String
  hospitalId     String?
  agencyId       String?
  status         ConsultationStatus @default(PENDING)
  requestedDate  DateTime?
  confirmedDate  DateTime?
  treatmentDate  DateTime?
  memo           String?
  createdAt      DateTime           @default(now())
  updatedAt      DateTime           @updatedAt

  patient  Patient   @relation(fields: [patientId], references: [id])
  hospital Hospital? @relation(fields: [hospitalId], references: [id])
  agency   Agency?   @relation(fields: [agencyId], references: [id])
}

enum ConsultationStatus {
  PENDING
  IN_PROGRESS
  CONFIRMED
  COMPLETED
  CANCELLED
}
```

### Agency

```prisma
model Agency {
  id             String      @id @default(uuid())
  name           String
  country        String
  contactEmail   String
  contactPhone   String?
  commissionRate Float       @default(0)
  status         AgencyStatus @default(ACTIVE)
  deletedAt      DateTime?
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt

  patients      Patient[]
  consultations Consultation[]
}

enum AgencyStatus {
  ACTIVE
  INACTIVE
}
```

### Notification

```prisma
model Notification {
  id        String           @id @default(uuid())
  adminId   String
  type      NotificationType
  title     String
  body      String
  isRead    Boolean          @default(false)
  refId     String?
  refType   String?
  createdAt DateTime         @default(now())

  admin Admin @relation(fields: [adminId], references: [id])
}

enum NotificationType {
  HOSPITAL_REGISTER
  CONSULTATION_NEW
  CONSULTATION_UPDATE
  AGENCY_REGISTER
  SYSTEM
}
```

---

## 8. 개발 로드맵

### Phase 1 — 핵심 기반 (4~6주)

**목표:** 인증 + 병원/환자 기본 CRUD 동작

- [ ] 프로젝트 세팅 (Next.js 16 + Prisma 7 + PostgreSQL)
- [ ] DB 스키마 마이그레이션
- [ ] JWT 인증 (로그인, 로그아웃, 갱신)
- [ ] 공통 레이아웃 컴포넌트 (Sidebar, TopBar)
- [ ] 병원 목록/등록/상태 변경
- [ ] 환자 목록/등록/수정
- [ ] 대시보드 기본 통계 카드
- [ ] S3 파일 업로드 연동

### Phase 2 — 상담 관리 (3~4주)

**목표:** 칸반 + 에이전시 + 알림 시스템

- [ ] 상담 칸반 CRUD
- [ ] 드래그&드롭 상태 이동 (`@dnd-kit`)
- [ ] 병원 배정 로직
- [ ] 에이전시 관리 CRUD
- [ ] 실시간 알림 (SSE)
- [ ] 이메일 발송 연동 (SendGrid)
- [ ] 알림 센터 페이지

### Phase 3 — 통계 + 설정 (3~4주)

**목표:** 데이터 분석 + 시스템 설정

- [ ] 통계 집계 API (Prisma aggregate)
- [ ] Chart.js / Recharts 동적 차트 연동
- [ ] 기간 필터 적용
- [ ] 권한 관리 UI 연동
- [ ] 시스템 설정 저장/조회
- [ ] 감사 로그
- [ ] CSV 내보내기

### Phase 4 — 고도화 (4~6주)

**목표:** UX 완성 + 성능 최적화

- [ ] 카카오맵 SDK 연동 (병원 검색 지도)
- [ ] 드래그 노출 순서 변경
- [ ] 수수료 정산 기능
- [ ] 반응형 완성 (1024px 이하)
- [ ] 접근성 (ARIA, 키보드 내비게이션)
- [ ] 다국어 지원 기반 (i18n)
- [ ] 성능 최적화 (React Query, 캐싱)

---

## 9. 기술 스택

### 권장 스택

| 레이어 | 기술 | 비고 |
|--------|------|------|
| **Frontend** | Next.js 16, React 19, TypeScript | App Router |
| **스타일** | Tailwind CSS 4 | 기존 CSS 변수 시스템 활용 |
| **폼** | react-hook-form + zod | 유효성 검사 |
| **상태 관리** | TanStack Query (React Query) | API 캐싱 |
| **드래그** | @dnd-kit | 칸반, 순서 변경 |
| **차트** | Chart.js 또는 Recharts | 통계 페이지 |
| **ORM** | Prisma 7 | PostgreSQL |
| **DB** | PostgreSQL | 메인 데이터 |
| **캐시** | Redis | 세션, 알림 큐 |
| **파일** | AWS S3 | 병원 이미지 등 |
| **이메일** | SendGrid / AWS SES | 알림 발송 |
| **지도** | 카카오맵 JavaScript SDK | 병원 검색 |
| **배포** | Vercel / AWS | — |

### 공통 CSS 시스템 활용

현재 `assets/css/common.css`에 정의된 CSS 변수를 Tailwind config에 등록하여 재사용:

```js
// tailwind.config.js
theme: {
  extend: {
    colors: {
      accent: '#3B43DB',
      'accent-light': '#EEEFFC',
      // ...
    }
  }
}
```

### 디렉토리 구조 (Next.js)

```
src/
├── app/
│   ├── (auth)/login/
│   ├── (admin)/
│   │   ├── dashboard/
│   │   ├── hospitals/
│   │   ├── patients/
│   │   ├── consultations/
│   │   ├── agencies/
│   │   ├── statistics/
│   │   └── settings/
│   └── api/           # Route Handlers
├── components/
│   ├── layout/        # Sidebar, TopBar
│   ├── ui/            # Button, Badge, Modal, etc.
│   └── domain/        # HospitalCard, PatientPanel, KanbanBoard, etc.
├── lib/
│   ├── prisma.ts
│   ├── auth.ts
│   └── s3.ts
└── types/
```

---

*보고서 HTML: `medi-w-dev-report.html` — 시각적 버전 참조*
