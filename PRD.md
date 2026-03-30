# PRD — 뱅크법률집사 (행원 전용 앱)

## 1. Product Overview

| 항목 | 내용 |
|------|------|
| **서비스명** | 뱅크법률집사 — 행원 (BankCasa Teller) |
| **한 줄 설명** | 신협·단위농협·새마을금고 행원이 AI 기반 법률 상담, 서류 자동화, 채권계산 등을 수행하는 전용 앱. bankcasa-demo와 동일 코드베이스에서 `role=teller`로 진입. |
| **대상 사용자** | 신협·단위농협·새마을금고 행원 |

## 2. Tech Stack

| 구분 | 기술 |
|------|------|
| 프레임워크 | React 19 + TypeScript |
| 빌드 도구 | Vite 8 |
| 스타일링 | Tailwind CSS |
| 라우팅 | HashRouter (react-router-dom) |
| 상태 관리 | Context API (AppProvider) |
| 저장소 | localStorage |
| 배포 | GitHub Pages (gh-pages 브랜치) |

> **참고:** bankcasa-demo와 동일 코드베이스. URL 파라미터 `role=teller`로 자동 진입하여 행원 전용 UI를 표시한다.

## 3. Architecture

```
SPA (bankcasa-demo 동일 코드베이스)
├── URL 진입: #/?role=teller&inst=xxx&name=xxx
├── AppProvider → role='teller' 설정
├── AppLayout
│   ├── 좌측 사이드바 (행원 전용 메뉴)
│   └── 메인 콘텐츠 영역
└── Pages/ (bankcasa-demo와 동일)
```

## 4. Pages & Routes

### 사이드바 메뉴 (행원 전용)

| 메뉴 | 경로 | 설명 |
|------|------|------|
| 집변 법률상담 | 외부 링크 | `homelawyer.kr` — 검정 강조 스타일, 새 탭 |
| 상황별 서류 작성 | `/consult` | AI 상담 → 서류 생성 플로우 진입 |
| 서류 자동화 | `/documents-catalog` | 8종 서류 카탈로그 카드 그리드 |
| 채권계산기 | `/calculator` | 원금+이자+지연손해금+인지대·송달료 |
| AI 법령·지침 검색 | `/regulation-search` | 법령/내규/지침 키워드 검색 |
| 상담 내역 | `/history` | 과거 상담 기록 조회 |
| 마이페이지 | `/mypage` | 사용자 정보 확인 |

### 전체 라우트 (bankcasa-demo 참조)

bankcasa-demo의 모든 라우트를 공유하되, 행원 역할에 해당하는 페이지만 사이드바에 노출된다.

## 5. Data Models

bankcasa-demo와 동일. 주요 타입:

```typescript
interface User {
  institution: string;
  name: string;
  role?: 'teller' | 'member' | 'chairman';
}

interface ConsultHistory {
  id: string;
  date: string;
  caseType: string;
  caseName: string;
  status: string;
  formData?: Record<string, any>;
  claimCalc?: ClaimCalculation;
}

interface CaseData {
  id: string;
  name: string;
  description: string;
  documents: string[];
  formFields: FormField[];
  procedureSteps: string[];
  documentTemplate: string;
}

type DecisionTreeNode =
  | { result: string }
  | { question: string; options: Record<string, DecisionTreeNode> };
```

## 6. Key Features

### 6.1 상황별 서류 작성 (핵심 기능)
- AI 의사결정 트리 기반 상담 → 적합한 서류 유형 자동 추천
- 채팅 모드 / 버튼 모드 토글
- 서류 생성 플로우: 상담 → 서류 안내 → 정보 입력 → 생성 중 → 미리보기 → 완료

### 6.2 서류 자동화 (8종)
임의경매 / 가압류 / 지급명령 / 추심명령 / 내용증명 / 강제경매 / 이의신청 / 출자금반환

### 6.3 채권계산기
- 원금 + 이자(약정이율) + 지연손해금(법정이율) + 인지대·송달료
- 소송촉진법 §3① 연 12%, 민법 §379 연 5%

### 6.4 AI 법령·지침 검색
- 키워드 기반 법령/내규/지침 검색
- 관련 법조문 표시

## 7. Design System

| 속성 | 값 |
|------|-----|
| 레이아웃 | 좌측 사이드바 + 메인 콘텐츠 (데스크탑 최적화) |
| 배경색 | `#ffffff` |
| 텍스트색 | `#000000` |
| 구분선 | `1px solid #e5e5e5` |
| 버튼 | `border-radius: 2px`, 검정/흰색만 |
| 폰트 | Pretendard |
| 집변 링크 | 검정 강조 스타일 (사이드바 최상단) |

## 8. External Integrations

| 서비스 | URL | 연동 방식 |
|--------|-----|-----------|
| 집변 (법률 상담) | `homelawyer.kr` | `window.open()` — 사이드바 최상단, 검정 강조 |
| 법령정보 | `law.go.kr` | 외부 링크 |

## 9. Deployment

| 항목 | 값 |
|------|-----|
| 호스팅 | GitHub Pages |
| 브랜치 | `gh-pages` |
| URL | `https://{owner}.github.io/bankcasa-teller/` |
| 진입 | URL 파라미터 `#/?role=teller&inst=xxx&name=xxx` |

## 10. Known Limitations

- **bankcasa-demo와 동일 코드베이스** — 별도 빌드가 아닌 URL 파라미터로 역할 분기
- **모든 데이터는 목업** — 실 API 연동 없음
- **AI 상담은 정적 의사결정 트리** — LLM 미연동
- **채권계산은 클라이언트 사이드 계산** — 서버 검증 없음
- **서류 템플릿은 하드코딩된 문자열 치환**
- **인증/인가 없음**
