# Menual — 뷰티 전문가 매칭 플랫폼

헤어, 패션, 메이크업, 피부 분야의 전문가와 고객을 연결하는 뷰티 컨설팅 서비스

---

## 📖 프로젝트 개요

Menual은 고객이 단계별 고민지를 작성해 뷰티 전문가에게 전달하면, 전문가가 솔루션지로 답하고 실시간 채팅으로 추가 상담까지 이어지는 플랫폼입니다.

---

## 🛠 기술 스택

### Development

<div align="left">
<img src="https://img.shields.io/badge/React_19-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
<img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" />
<img src="https://img.shields.io/badge/Vite_7-646CFF?style=for-the-badge&logo=vite&logoColor=white" />
<img src="https://img.shields.io/badge/TailwindCSS_v4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" />
<img src="https://img.shields.io/badge/Zustand-443E38?style=for-the-badge&logo=react&logoColor=white" />
<img src="https://img.shields.io/badge/STOMP.js-35495E?style=for-the-badge&logo=socket.io&logoColor=white" />
<img src="https://img.shields.io/badge/Axios-5A29E4?style=for-the-badge&logo=axios&logoColor=white" />
<img src="https://img.shields.io/badge/Zod-3E67B1?style=for-the-badge&logo=zod&logoColor=white" />
</div>

### Deployment

<div align="left">
<img src="https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white" />
<img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" />
</div>

---

## 📆 프로젝트 기간

- 개발 기간: `2025.09 - 2026.02`

---

# 내가 구현한 기능

> 팀 프로젝트 **Menual** 내에서 직접 구현한 파트의 포트폴리오 문서입니다.

---

## 1. 배포 자동화 (CI/CD)

**GitHub Actions + Vercel 기반 자동 배포 파이프라인 구성**

`develop` 브랜치에 Push가 발생하면 별도의 조작 없이 프로덕션까지 자동으로 배포됩니다.

- GitHub Actions에서 빌드 후 `cpina/github-action-push-to-another-repository`로 빌드 결과물을 배포 전용 레포지토리에 자동 Push
- Vercel이 배포 레포를 감지해 즉시 프로덕션 반영
- 커밋 메시지를 배포 레포의 커밋에 그대로 전달해 배포 이력 추적 가능

```yaml
# .github/workflows/deploy.yml
- name: Pushes to another repository
  uses: cpina/github-action-push-to-another-repository@main
  env:
    API_TOKEN_GITHUB: ${{ secrets.AUTO_ACTIONS }}
  with:
    source-directory: "output"
    destination-github-username: dragunshin
    destination-repository-name: ge-fe
    commit-message: ${{ github.event.commits[0].message }}
    target-branch: develop
```

---

## 2. 실시간 채팅 (Chat)

**STOMP over SockJS 기반 WebSocket 실시간 채팅 시스템**

### useStompClient — 모듈 싱글톤 연결 관리

여러 컴포넌트가 동시에 마운트되어도 WebSocket 연결이 단 하나만 유지되도록 모듈 레벨 싱글톤 패턴을 적용했습니다.

- 마운트 카운터(`mounts`)로 연결 생명주기 관리 → 모든 컴포넌트 언마운트 시에만 연결 해제
- `onConnect` 프레임의 `user-name` 헤더를 파싱해 본인 `userId`를 추출 → 메시지 말풍선 좌우 구분에 활용
- 재연결 시 `onConnect`에서 밀린 구독을 자동 flush → 네트워크 복구 후 메시지 수신 즉시 재개
- `ws://` / `wss://` 스킴을 `http://` / `https://`로 자동 변환 (SockJS 요구사항 처리)

```typescript
// onConnect에서 본인 userId 파싱
onConnect: (frame: IFrame) => {
  emitReady(true);
  const myId = parseMyUserIdFromHeaders(frame.headers as Record<string, string>);
  if (myId != null) emitMyUserId(myId);

  // 재연결 시 밀린 구독 flush
  for (const r of subs) {
    if (!r.active) continue;
    if (!r.sub) r.sub = client.subscribe(r.destination, r.handler);
  }
},
```

### useChatRoom — 채팅방 메시지 상태 관리

- 히스토리 로드 실패(403)가 실시간 연결을 막지 않도록 독립적인 `useEffect`로 분리
- `roomIdRef`로 콜백 내 stale 클로저 방지 → 룸 ID 변경 시에도 안전하게 구독
- `functional update` + `messageId` 중복 체크로 서버 에코 메시지 중복 렌더링 방지

```typescript
// functional update로 stale 클로저 없이 중복 메시지 제거
setMessages((prev) => {
  if (prev.some((m) => m.messageId === incoming.messageId)) return prev;
  return [...prev, incoming];
});
```

---

## 3. 고민지 플로우 — 헤어 (Hair Flow)

**URL 쿼리 파라미터 기반 7단계 멀티 스텝 폼**

헤어 상담에 필요한 정보(사진 4종 + 얼굴형 + 원하는 이미지 + 질문)를 단계별로 수집합니다.

| 단계 | 내용                          |
| ---- | ----------------------------- |
| 1/7  | 평상시 헤어스타일 사진 업로드 |
| 2/7  | 머리를 올린 정면 사진 업로드  |
| 3/7  | 측면 사진 업로드              |
| 4/7  | 원하는 이미지 사진 업로드     |
| 5/7  | 얼굴형 선택                   |
| 6/7  | 원하는 헤어 스타일 선택       |
| 7/7  | 추가 질문 작성                |

- `?step=N&reservationId=X` 쿼리 파라미터로 단계 관리 → 새로고침, 공유 URL에도 상태 유지
- `useLayoutEffect` + `requestAnimationFrame`으로 단계 전환마다 스크롤이 상단으로 이동
- `useHairSetupStore` (Zustand)로 단계 간 사진 키(S3 경로) 보존
- S3 Presigned URL로 사진을 직접 업로드 → 서버 부하 없이 대용량 이미지 처리

```typescript
// 단계 전환 시 스크롤 탑 보장
useLayoutEffect(() => {
  requestAnimationFrame(() => {
    window.scrollTo({ top: 0, left: 0, behavior: "auto" });
    (document.scrollingElement ?? document.documentElement).scrollTop = 0;
  });
}, [loc.key, step]);
```

---

## 4. S3 이미지 업로드 — AbortController 기반 안전한 프리뷰 관리

**문제 상황: 서버 오류 시 프리뷰가 남아 "업로드된 것처럼" 보이는 UX 오류**

사진을 선택하면 로컬 `Object URL`로 프리뷰를 즉시 보여줍니다. 그런데 이후 S3 업로드가 실패하면, 프리뷰는 그대로 보이는 채 S3 키만 없는 상태가 됩니다. 사용자는 업로드가 된 줄 알고 다음 단계로 넘어가지만 실제로는 이미지가 저장되지 않은 문제가 발생했습니다.

### 해결: 업로드 결과에 따른 프리뷰 생명주기 명확히 분리

**단일 사진 업로드 (`SinglePhotoField`)**

- 파일 선택 즉시 `URL.createObjectURL()`로 로컬 프리뷰를 표시하고 S3 업로드 시작
- `AbortController`를 `abortRef`로 보관 → 유저가 X 버튼을 누르거나, 새 파일을 선택하거나, 컴포넌트가 언마운트될 때 `abort()` 호출로 진행 중인 presign-PUT 요청을 즉시 취소
- `seqRef` (업로드 순서 카운터)로 새 업로드가 시작된 뒤 도착한 이전 응답을 무시 → 빠르게 사진을 바꿔도 stale 결과가 Zustand에 저장되지 않음
- **S3 업로드 실패 시**: `URL.revokeObjectURL()`로 프리뷰를 즉시 제거 → 사용자가 업로드가 완료된 것으로 오인하는 상황 방지
- presign 요청과 S3 PUT 요청 모두 동일한 `signal`을 전달해 중단 시 네트워크 요청이 두 단계 모두 취소됨

```typescript
const onChange = async (file: File) => {
  handleRemove(); // 이전 업로드 abort + 프리뷰 초기화

  const localUrl = URL.createObjectURL(file);
  setPreviewUrl(localUrl); // 로컬 프리뷰 즉시 표시

  const controller = new AbortController();
  abortRef.current = controller;
  const mySeq = ++seqRef.current;

  try {
    const { key } = await uploadImageViaPresign({
      file,
      resourceType,
      resourceId,
      imageType,
      signal: controller.signal, // presign + PUT 모두 동일 signal 전달
    });

    if (seqRef.current !== mySeq) return; // 새 업로드가 시작됐으면 결과 무시
    onUploadedKey(key); // 성공 시에만 Zustand에 S3 키 저장
  } catch (e) {
    if (controller.signal.aborted) return; // abort로 인한 에러는 무시
    clearLocalPreview(); // 실패 시 프리뷰 제거 → 오해 방지
  } finally {
    if (seqRef.current === mySeq) setIsUploading(false);
  }
};
```

**다중 사진 업로드 (`MultiPhotoPicker`)**

- 생성한 모든 `Object URL`을 `objectUrlsRef`(Set)에 등록해 누락 없이 추적
- 업로드 실패 시 해당 `Object URL`만 즉시 revoke + Set에서 제거 → 프리뷰 사라짐
- 컴포넌트 언마운트 시 Set에 남은 모든 URL을 일괄 revoke → 메모리 누수 방지

```typescript
useEffect(() => {
  return () => {
    objectUrlsRef.current.forEach((url) => URL.revokeObjectURL(url));
    objectUrlsRef.current.clear();
  };
}, []);

const addFile = async (file: File) => {
  const localUrl = URL.createObjectURL(file);
  objectUrlsRef.current.add(localUrl);

  try {
    const { key } = await uploadImageViaPresign({ file, ... });
    setPreviews((p) => [...p, { key, previewUrl: localUrl }]); // 성공 시에만 프리뷰 등록
  } catch {
    URL.revokeObjectURL(localUrl);
    objectUrlsRef.current.delete(localUrl); // 실패 시 프리뷰 즉시 제거
  }
};
```

---

## 5. 솔루션지 (Solution)

**React Quill 기반 전문가 솔루션 작성 에디터**

- 헤어 / 패션 카테고리별 기본 템플릿 HTML을 Quill `value`로 주입해 전문가가 항목별로 바로 작성 가능
- 이미지 삽입 시 S3 Presigned URL로 직접 업로드 후 이미지 URL을 Quill에 삽입
- 고객의 고민지(설문 응답)를 상단에 함께 렌더링해 전문가가 컨텍스트를 보며 작성 가능
- 고객은 `/consultations/:consultationId/solution`에서 솔루션지 열람 및 내 솔루션 목록 조회

---

## 6. 마이페이지 (MyPage)

**일반 사용자 / 전문가 역할별로 분기되는 마이페이지**

| 페이지            | 내용                                             |
| ----------------- | ------------------------------------------------ |
| 마이페이지 메인   | 유저 타입 감지 후 일반 / 전문가 메뉴 분기 렌더링 |
| 찜 목록           | 좋아요한 전문가 목록 조회                        |
| 포인트 내역       | 포인트 적립 / 사용 내역                          |
| 결제 내역         | 결제 내역 목록                                   |
| 예약 내역         | 예약 상태별 목록 조회                            |
| 내 리뷰           | 작성한 리뷰 목록 조회                            |
| 리뷰 작성         | 상담 완료 건에 대한 별점 + 텍스트 리뷰 작성      |
| 전문가 자기소개   | React Quill로 자기소개 편집                      |
| 전문가 포트폴리오 | 포트폴리오 목록 관리 (위 4번 참고)               |
| 전문가 일정 설정  | 상담 유형 / 가격 설정 (위 4번 참고)              |
| 전문가 상담 내역  | 진행한 상담 이력 조회                            |

---

## 기술적 도전 & 해결

| 문제                                                                 | 해결 방법                                                                                  |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 여러 컴포넌트에서 동시에 WebSocket을 마운트하면 중복 연결 발생       | 모듈 레벨 싱글톤 + 마운트 카운터로 연결 1개 유지                                           |
| 재연결 후 구독이 사라져 메시지 미수신                                | `onConnect`에서 활성 구독 목록을 flush해 자동 재구독                                       |
| 소켓 콜백 내 `roomId`가 stale 클로저로 이전 값 참조                  | `useRef`로 최신 값 동기화, `functional update`로 `prev` 사용                               |
| 서버 에코 메시지로 인한 메시지 중복 렌더링                           | `messageId` 기반 중복 체크                                                                 |
| 단계 전환 시 스크롤 위치가 이전 단계의 위치에 머무름                 | `useLayoutEffect` + `requestAnimationFrame`으로 렌더 후 스크롤 탑 보장                     |
| S3 업로드 실패 시 프리뷰가 남아 업로드 완료로 오인                   | 업로드 실패 즉시 `URL.revokeObjectURL()`로 프리뷰 제거, S3 키는 성공 시에만 Zustand에 저장 |
| 사진을 빠르게 교체하면 이전 업로드 결과가 늦게 도착해 잘못된 키 저장 | `seqRef` 카운터로 stale 응답 감지 후 무시, `AbortController`로 이전 요청 즉시 취소         |
| 컴포넌트 언마운트 시 Object URL이 메모리에 누적                      | `objectUrlsRef`(Set)로 생성한 모든 URL 추적 → 언마운트 시 일괄 revoke                      |

---
