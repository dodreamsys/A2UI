# A2UI 프로젝트 분석 및 검색 보조 에이전트 아이디어 연결

## 1. A2UI 프로젝트 개요

### 1.1 A2UI란?
**A2UI (Agent-to-User Interface)**는 Google에서 개발한 오픈소스 프로젝트로, AI 에이전트가 사용자에게 **동적인 리치 UI**를 생성하고 전달할 수 있게 해주는 프로토콜 및 렌더러 세트입니다.

### 1.2 핵심 철학
```
"Safe like data, but expressive like code"
(데이터처럼 안전하지만, 코드처럼 표현력 있는)
```

- **Security First**: LLM이 생성하는 것은 실행 코드가 아닌 선언적 데이터 포맷
- **LLM-Friendly**: 평면(flat) 구조로 LLM이 쉽게 생성 가능
- **Framework-Agnostic**: Flutter, Web, React 등 어떤 플랫폼에서도 렌더링 가능
- **Incrementally Updateable**: 점진적 업데이트로 빠른 UX 제공

---

## 2. A2UI 아키텍처

### 2.1 데이터 흐름
```
Agent (LLM) → A2UI JSON 생성 → Transport (SSE/WebSocket/A2A)
                                        ↓
Native UI ← Renderer ← Message Parser ← Client
```

### 2.2 핵심 메시지 타입
| 메시지 | 역할 |
|--------|------|
| `surfaceUpdate` | UI 컴포넌트 구조 정의/업데이트 |
| `dataModelUpdate` | 애플리케이션 상태(데이터) 업데이트 |
| `beginRendering` | 렌더링 시작 신호 |
| `deleteSurface` | UI surface 제거 |
| `userAction` | 사용자 상호작용 전송 (클라이언트 → 서버) |

### 2.3 Adjacency List 모델
A2UI의 핵심 설계 중 하나로, **중첩된 트리 대신 평면 리스트**를 사용합니다:

```json
{
  "components": [
    {"id": "root", "component": {"Column": {"children": {"explicitList": ["title", "button"]}}}},
    {"id": "title", "component": {"Text": {"text": {"literalString": "Hello"}}}},
    {"id": "button", "component": {"Button": {"child": "btn-text", "action": {"name": "click"}}}}
  ]
}
```

**장점:**
- LLM이 한 번에 완벽한 중첩 구조를 생성할 필요 없음
- 어떤 컴포넌트든 ID로 직접 업데이트 가능
- 스트리밍으로 점진적 전송 가능

### 2.4 Data Binding
UI 구조와 데이터를 분리하여 **Reactive Updates** 가능:

```json
// 데이터 바인딩
{"text": {"path": "/user/name"}}

// 데이터 업데이트만으로 UI 자동 갱신
{"dataModelUpdate": {"path": "/user", "contents": [{"key": "name", "valueString": "Bob"}]}}
```

---

## 3. 프로젝트 구조

```
A2UI/
├── specification/     # JSON Schema 및 프로토콜 스펙
│   ├── 0.8/          # v0.8 스펙
│   └── 0.9/          # v0.9 스펙 (진행 중)
├── renderers/         # 클라이언트 렌더러
│   ├── lit/          # Web Components (Lit)
│   └── angular/      # Angular 렌더러
├── samples/           # 예제
│   ├── agent/        # Python 에이전트 예제
│   └── client/       # 클라이언트 예제
├── a2a_agents/        # A2A 프로토콜 통합
├── tools/             # 개발 도구
│   ├── editor/       # A2UI 에디터
│   └── inspector/    # 디버깅 도구
└── docs/              # 문서
```

### 3.1 표준 컴포넌트 카탈로그
| 카테고리 | 컴포넌트 |
|----------|----------|
| **Layout** | Row, Column, List |
| **Display** | Text, Image, Icon, Video, Divider |
| **Interactive** | Button, TextField, CheckBox, DateTimeInput, Slider, MultipleChoice |
| **Container** | Card, Tabs, Modal |

---

## 4. 당신의 아이디어와 A2UI 비교

### 4.1 당신의 아이디어
> "사용자와 AI가 같은 Next.js 화면을 보고 소통하게 하고 싶어. AI도 조작할 수 있지만 사람도 조작할 수 있게."

### 4.2 A2UI의 접근 방식
| 측면 | A2UI | 당신의 아이디어 |
|------|------|-----------------|
| **UI 생성 주체** | AI가 UI를 생성하여 사용자에게 전달 | AI와 사용자가 **동일한 UI**를 공유하며 양방향 조작 |
| **공유 상태** | 단방향 (Agent → Client) | **양방향** (Agent ↔ User) |
| **화면 공유** | 별도 surface로 분리 | **동일한 화면**을 실시간 공유 |
| **조작 권한** | 사용자는 Agent가 정의한 인터랙션만 가능 | 양쪽 모두 **자유롭게 조작** |

### 4.3 핵심 차이점
```
A2UI:        Agent ──생성──> UI <──반응── User
당신의 아이디어: Agent <──조작──> 공유 UI <──조작──> User
```

---

## 5. A2UI에서 얻을 수 있는 아이디어

### 5.1 데이터 바인딩 아키텍처
**활용 포인트:** UI 구조와 데이터의 분리
```javascript
// 공유 상태 모델
const sharedState = {
  searchQuery: "",
  results: [],
  selectedItem: null,
  cursor: { x: 0, y: 0 } // AI 커서 위치
};

// UI는 이 상태에 바인딩
<SearchInput value={sharedState.searchQuery} />
<ResultList items={sharedState.results} />
<AICursor position={sharedState.cursor} />
```

### 5.2 Surface 개념의 확장
**활용 포인트:** 독립적인 UI 영역 관리

```javascript
// 여러 독립 영역을 관리
const surfaces = {
  "main-content": { components: [...], data: {...} },
  "ai-suggestions": { components: [...], data: {...} },
  "chat-panel": { components: [...], data: {...} }
};
```

### 5.3 Adjacency List 모델
**활용 포인트:** 협업 편집에 유리한 구조

```javascript
// 평면 구조로 충돌 최소화
const components = new Map([
  ["search-box", { type: "input", props: {...}, modifiedBy: "user" }],
  ["result-1", { type: "card", props: {...}, modifiedBy: "ai" }],
  ["result-2", { type: "card", props: {...}, modifiedBy: "ai" }]
]);

// 특정 컴포넌트만 업데이트
function updateComponent(id, patch, actor) {
  const component = components.get(id);
  components.set(id, { ...component, ...patch, modifiedBy: actor });
}
```

### 5.4 Action 시스템
**활용 포인트:** 양방향 이벤트 처리

```javascript
// A2UI의 Action 구조를 확장
const action = {
  name: "select_result",
  actor: "user" | "ai",  // 누가 실행했는지
  context: {
    resultId: "result-123",
    timestamp: "2025-01-15T10:30:00Z"
  }
};

// AI와 사용자 모두 같은 액션 시스템 사용
function handleAction(action) {
  if (action.name === "select_result") {
    highlightResult(action.context.resultId);
    // 상대방에게 알림
    broadcastToOther(action);
  }
}
```

### 5.5 점진적 렌더링
**활용 포인트:** AI 작업의 실시간 시각화

```javascript
// AI가 검색 결과를 하나씩 추가하는 것을 실시간으로 표시
async function* aiSearch(query) {
  for await (const result of searchStream(query)) {
    yield { surfaceUpdate: { components: [newResultCard(result)] } };
    // 사용자는 AI가 결과를 추가하는 것을 실시간으로 봄
  }
}
```

---

## 6. 구현 제안 아키텍처

### 6.1 공유 상태 레이어
```
┌─────────────────────────────────────────────────┐
│              Shared State Manager               │
│  (CRDT 또는 OT 기반 실시간 동기화)                │
├─────────────────────────────────────────────────┤
│  ┌──────────────┐      ┌──────────────┐         │
│  │  User Input  │      │   AI Input   │         │
│  │  Controller  │      │  Controller  │         │
│  └──────┬───────┘      └──────┬───────┘         │
│         │                     │                 │
│         ▼                     ▼                 │
│  ┌─────────────────────────────────────────┐    │
│  │            Shared Data Model            │    │
│  │  - UI Components (Adjacency List)       │    │
│  │  - Application State (JSON)             │    │
│  │  - Cursor Positions                     │    │
│  │  - Selection States                     │    │
│  └─────────────────────────────────────────┘    │
│                      │                          │
│                      ▼                          │
│  ┌─────────────────────────────────────────┐    │
│  │           Next.js Renderer              │    │
│  │  (React Components + Real-time Sync)    │    │
│  └─────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### 6.2 핵심 구성요소

#### (1) 공유 상태 동기화
```typescript
// Yjs 또는 Automerge 활용
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

const ydoc = new Y.Doc();
const yComponents = ydoc.getMap('components');
const yDataModel = ydoc.getMap('dataModel');

// AI와 User 모두 같은 문서 편집
yComponents.set('search-box', { type: 'TextField', text: '검색어' });
```

#### (2) AI 존재 표시
```typescript
// AI의 현재 작업 상태를 시각화
interface AIPresence {
  cursor: { x: number; y: number };
  currentAction: 'searching' | 'analyzing' | 'selecting' | 'idle';
  focusedElement: string | null;
  thinkingMessage: string | null;
}

// 사용자 UI에 AI 커서/상태 오버레이
<AICursorOverlay presence={aiPresence} />
<AIStatusIndicator action={aiPresence.currentAction} />
```

#### (3) 권한 및 충돌 해결
```typescript
// 동시 편집 충돌 해결 전략
interface EditPolicy {
  // 특정 요소에 대한 편집 우선권
  getEditPriority(elementId: string, actor: 'user' | 'ai'): number;

  // 충돌 발생 시 처리
  resolveConflict(
    element: Component,
    userEdit: Patch,
    aiEdit: Patch
  ): Component;
}

// 예: 사용자가 현재 조작 중인 요소는 AI가 수정하지 않음
const policy: EditPolicy = {
  getEditPriority(elementId, actor) {
    if (userSelection.includes(elementId) && actor === 'ai') {
      return -1; // AI 편집 차단
    }
    return actor === 'user' ? 10 : 5; // 사용자 우선
  }
};
```

#### (4) 통합 액션 핸들러
```typescript
// 양쪽에서 발생하는 액션을 통합 처리
type UnifiedAction = {
  type: string;
  payload: any;
  actor: 'user' | 'ai';
  timestamp: number;
};

function handleUnifiedAction(action: UnifiedAction) {
  // 1. 상태 업데이트
  updateSharedState(action);

  // 2. 상대방에게 브로드캐스트
  broadcastAction(action);

  // 3. AI/User 특정 후처리
  if (action.actor === 'user') {
    notifyAI(action); // AI에게 사용자 행동 알림
  } else {
    showAIActionFeedback(action); // 사용자에게 AI 행동 표시
  }
}
```

---

## 7. 추천 기술 스택

### 7.1 프론트엔드
- **Next.js 14+** (App Router)
- **React 18+** (Concurrent Features 활용)
- **Yjs** 또는 **Automerge** (CRDT 기반 실시간 동기화)
- **Socket.io** 또는 **PartyKit** (실시간 통신)

### 7.2 백엔드
- **Next.js API Routes** 또는 **별도 WebSocket 서버**
- **LangChain** 또는 **Vercel AI SDK** (AI 통합)
- **Redis** (상태 저장 및 Pub/Sub)

### 7.3 AI 연동
- **OpenAI API** / **Anthropic API** / **Gemini API**
- **Streaming Response** 필수
- **Function Calling** 활용하여 UI 조작 명령 구조화

---

## 8. 다음 단계 제안

1. **프로토타입 구현**: 간단한 검색 UI에서 AI 커서 공유만 먼저 구현
2. **CRDT 라이브러리 선택**: Yjs vs Automerge 비교 테스트
3. **AI 행동 시각화 디자인**: AI가 무엇을 하고 있는지 사용자에게 어떻게 보여줄지
4. **충돌 해결 정책 정의**: 동시 편집 시 우선순위 규칙

---

## 9. 참고 자료

- [A2UI GitHub](https://github.com/google/A2UI)
- [A2A Protocol](https://a2a-protocol.org)
- [Yjs CRDT Library](https://yjs.dev)
- [Automerge](https://automerge.org)
- [PartyKit](https://partykit.io)
- [Vercel AI SDK](https://sdk.vercel.ai)

---

*작성일: 2025-12-19*
