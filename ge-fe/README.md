# Menual - 뷰티 전문가 매칭 플랫폼 (프론트엔드)

> 헤어 · 메이크업 · 패션 · 피부 분야의 전문가와 고객을 연결하는 O2O 뷰티 컨설팅 서비스

---

## 프로젝트 소개

Menual은 고객이 뷰티 전문가에게 헤어·패션 고민을 접수하고, 전문가가 솔루션을 제공한 뒤 실시간 채팅으로 추가 상담까지 이어지는 플랫폼입니다.  
고객은 단계별 플로우를 통해 사진·취향 정보를 제출하고, 전문가는 포트폴리오·일정을 직접 관리할 수 있습니다.

---

## 기술 스택

| 분류 | 기술 |
|---|---|
| 프레임워크 | React 19 + TypeScript |
| 번들러 | Vite 7 |
| 스타일링 | Tailwind CSS v4 |
| 라우팅 | React Router v7 |
| 상태 관리 | Zustand |
| HTTP 통신 | Axios |
| 실시간 통신 | STOMP.js + SockJS (WebSocket) |
| UI 컴포넌트 | Radix UI / shadcn-ui |
| 폼 검증 | Zod |
| 리치 에디터 | React Quill |
| 날짜 처리 | date-fns, React Day Picker |
| 파일 업로드 | AWS S3 Presigned URL |
| 배포 | Vercel + GitHub Actions |

---

## 주요 기능

### 인증 (Auth)
- 이메일 회원가입 / 로그인 (이메일 인증 포함)
- 카카오 소셜 로그인 (OAuth 콜백 처리)
- JWT 기반 인증 + HttpOnly 쿠키 관리
- Zustand Auth Store로 전역 인증 상태 관리
- 401/403 응답 시 Axios 인터셉터에서 자동 로그아웃

### 고민지 플로우 — 헤어 (Hair Flow)
4단계 멀티 스텝 폼으로 헤어 상담 정보를 수집합니다.

| 단계 | 내용 |
|---|---|
| Step 1 | 측면 사진 업로드 (S3 Presigned URL) |
| Step 2 | 얼굴형 선택 |
| Step 3 | 원하는 이미지 사진 업로드 |
| Step 4 | 추가 질문 작성 |

- URL 쿼리 파라미터(`?step=N&reservationId=X`)로 단계 관리 → 새로고침·공유에도 상태 유지
- `useLayoutEffect` + `requestAnimationFrame`으로 단계 전환 시 자동 스크롤 탑

### 고민지 플로우 — 패션 (Fashion Flow)
9단계 멀티 스텝 폼으로 패션 상담 정보를 수집합니다.

| 단계 | 내용 |
|---|---|
| Step 1 | 안내 가이드 |
| Step 2 | 측면 사진 업로드 |
| Step 3 | 착장 사진 업로드 |
| Step 4 | 체형 사이즈 입력 |
| Step 5 | 이미지 스타일 선택 |
| Step 6 | 스타일 선호도 선택 |
| Step 7 | 체형 콤플렉스 선택 |
| Step 8 | 아이템 예산 입력 |
| Step 9 | 목적 선택 |

### 실시간 채팅 (Chat)
- STOMP over SockJS 기반 WebSocket 실시간 채팅
- 모듈 싱글톤 패턴으로 중복 연결 방지 (마운트 카운터로 연결 생명주기 관리)
- `onConnect` 프레임에서 `user-name` 헤더를 파싱해 본인 userId 추출
- 재연결 시 밀린 구독 자동 flush
- 채팅방 목록 / 채팅방 입장 / 메시지 전송

### 전문가 시스템 (Expert)
- 카테고리별 전문가 목록 · 상세 프로필 조회
- 포트폴리오 추가 · 대표 지정 (S3 이미지 업로드 포함)
- 전문가 자기소개 편집 (React Quill 리치 에디터)
- 상담 일정 설정 · 수정 (React Day Picker 캘린더)
- 상담 내역 조회

### 예약 · 결제 (Reservation & Payment)
- 달력 기반 예약 슬롯 선택 (Bottom Sheet)
- 예약 유형 선택 (화상 / 채팅)
- 결제 주문 · 완료 페이지
- 예약 내역 / 결제 내역 조회

### 솔루션지 (Solution)
- 전문가 솔루션 작성 (React Quill 리치 에디터)
- 고객 솔루션 열람 (고민지 → 솔루션지 연결)
- 내 솔루션 목록 조회

### 탐색 · 카테고리 (Explore & Category)
- 카테고리별 랜딩 (헤어 / 메이크업 / 패션 / 피부)
- 전문가 목록 · 베스트 리뷰 목록
- 전문가 포트폴리오 갤러리

### 마이페이지 (MyPage)
- 찜 목록, 포인트 내역, 리뷰 작성 · 내 리뷰 목록
- 전문가 전용: 포트폴리오 관리, 소개 편집, 일정 설정, 상담 내역

---

## 프로젝트 구조

```
src/
├── api/           # 도메인별 API 함수 (chat, expert, hairFlow, ...)
├── components/    # 공통 컴포넌트 (auth-guard, UI primitives)
├── hooks/         # 커스텀 훅 (useChatRoom, useStompClient, ...)
├── lib/
│   ├── api/       # Axios 클라이언트 + 에러 핸들러
│   ├── schemas/   # Zod 스키마
│   └── utils/     # 유틸리티 함수
├── pages/         # 페이지 컴포넌트 (라우트 기반)
├── services/      # 비즈니스 로직 서비스 레이어
├── stores/        # Zustand 전역 스토어
├── types/         # 공통 타입 정의
└── router/        # React Router 라우트 설정
```

---

## 실행 방법

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# 빌드
npm run build
```

### 환경 변수 (.env)

```
VITE_API_BASE_URL=https://api.menual.site/api
VITE_WS_URL=https://api.menual.site/ws/chat
```

---

## 배포

`develop` 브랜치에 push 시 GitHub Actions가 빌드 결과물을 배포 레포지토리로 자동 push → Vercel에서 자동 배포됩니다.

---

## 담당 영역

- 전체 프론트엔드 아키텍처 설계 및 구현
- 인증 플로우 (이메일 인증, 카카오 소셜 로그인)
- 헤어 · 패션 고민지 멀티 스텝 플로우
- STOMP/SockJS 기반 실시간 채팅 시스템
- 전문가 포트폴리오 · 일정 관리 페이지
- Axios 인터셉터 기반 공통 에러 처리
- GitHub Actions CI/CD 파이프라인 구성
