# Harness 분석 정리 (한국어)

> 이 문서는 Harness 저장소를 처음 접한 사람을 위한 **한국어 요약 + 활용 가이드**입니다.
> 코드 자체를 바꾸지 않는 문서이며, 원본 프로젝트의 공식 문서를 대체하지 않습니다.

## 관련 링크

| 구분 | 주소 |
| --- | --- |
| 원본 저장소 (upstream) | https://github.com/awizemann/harness |
| 이 저장소 (fork) | https://github.com/bmshin94/harness |
| 랜딩 페이지 | https://awizemann.github.io/harness/ |
| 위키 | https://github.com/awizemann/harness/wiki |
| 릴리즈 | https://github.com/awizemann/harness/releases/latest |
| 라이선스 | MIT — Copyright (c) 2026 Alan Wizemann |

---

## 1. Harness가 뭐야?

한 줄 요약:

> **평범한 말로 "목표"와 "페르소나"를 적어주면, AI 에이전트가 실제 사용자처럼 앱을 직접 조작해보고
> "어디가 불편한지" 리포트를 뽑아주는 macOS 네이티브 개발자 도구.**

입력 예시:

- **목표:** "회원가입하고 첫 번째 리스트 만들기"
- **페르소나:** "이 앱을 처음 보는 사용자"

그러면 에이전트가 스크린샷을 읽고 → 클릭하고 → 입력하고 → 스크롤하면서 목표를 추구합니다.
중간에 막히거나 헷갈리는 지점(dead end, 모호한 라벨, 반응 없는 컨트롤)을 **friction 이벤트**로 기록합니다.

### 일반 UI 테스트와의 차이

| 일반 UI 테스트 (XCTest 등) | Harness |
| --- | --- |
| "버튼 A → 버튼 B" 대본대로만 실행 | 목표만 주면 경로는 스스로 탐색 |
| 통과 / 실패만 보고 | **어디서 헤맸는지(friction)** 까지 보고 |
| UI가 바뀌면 테스트가 깨짐 | 화면을 보고 판단하므로 상대적으로 유연 |

### 실행 1회당 나오는 결과물 3가지

1. **목표를 달성했는가** — 성공 / 실패 / 막힘 + 요약
2. **어떤 경로로 갔는가** — 화면 + 동작 순서 (리플레이 가능)
3. **어디서 마찰이 있었는가** — 타임스탬프가 찍힌 friction 이벤트 목록

---

## 2. 지원하는 3가지 타겟

| 대상 | 구동 방식 |
| --- | --- |
| **iOS 시뮬레이터** | `xcodebuild`로 빌드 → `simctl`로 boot/install/launch → WebDriverAgent로 입력 |
| **macOS 앱** | 접근성 API(`AXPress` / `AXSetValue`) 우선, 이후 `CGEvent.postToPid`. **실제 포인터가 움직이지 않고 포커스도 뺏지 않음** |
| **웹 앱** | 내장 `WKWebView` (기본 1280×1600) + JS 합성 이벤트 + `takeSnapshot` |

Application 단위로 종류를 한 번 선언하면, 에이전트의 도구 스키마와 시스템 프롬프트가 플랫폼별로 재구성됩니다.
실행 기록 / 리플레이 / friction 리포트는 플랫폼 중립입니다.

---

## 3. 폴더 구조

```
harness/
├── Harness/            # 메인 macOS 앱 (Swift 6 + SwiftUI, 89개 파일)
│   ├── App/            #   앱 진입점, AppCoordinator, FirstRunWizard
│   ├── Core/           #   모델, 경로 상수
│   ├── Domain/         #   AgentLoop 등 도메인 로직
│   ├── Services/       #   XcodeBuilder, SimulatorDriver, RunLogger,
│   │                   #   ProcessRunner, KeychainStore,
│   │                   #   ClaudeClient / OpenAIClient / GeminiClient / OllamaClient
│   ├── Features/       #   화면별 MVVM 묶음
│   │   ├── GoalInput/          # 목표 입력
│   │   ├── RunSession/         # 실행 중 라이브 화면
│   │   ├── RunHistory/         # 지난 실행 기록
│   │   ├── RunReplay/          # 리플레이
│   │   ├── FrictionReport/     # 마찰 리포트
│   │   ├── Personas/           # 페르소나 관리
│   │   ├── Applications/       # 테스트 대상 등록
│   │   ├── Actions/            # 재사용 가능한 작업 프롬프트
│   │   ├── AgentSessions/      # 에이전트 세션
│   │   └── Settings/           # 설정 (API 키, 로컬 모델)
│   ├── Platforms/      #   iOS / MacOS / Web 드라이버
│   ├── Tools/          #   에이전트 도구 스키마
│   └── UISessions/     #   스텝 단위 UI 세션
├── HarnessMCP/         # MCP 서버 (13개 파일) — 외부 에이전트 연결용
├── HarnessCLI/         # 터미널 드라이버 (6개 파일)
├── HarnessDesign/      # 디자인 시스템 (29개 파일)
├── Tests/              # 유닛 테스트 (49개 파일)
├── standards/          # 개발 표준 14개 문서
├── docs/               # ARCHITECTURE.md, ROADMAP.md, PROMPTS/
├── wiki/               # 위키 33페이지
├── releases/           # v0.1.0 ~ v0.8.4 릴리즈 노트
├── site/               # 랜딩 페이지
├── vendor/             # WebDriverAgent (git submodule)
├── .memory/            # Memophant — git에 저장되는 에이전트 메모리
├── project.yml         # xcodegen 설정 (Xcode 프로젝트를 여기서 생성)
└── CLAUDE.md           # 에이전트 페르소나 가이드
```

핵심 아키텍처 한 줄: **엔진 하나(RunCoordinator + 드라이버)를 GUI / CLI / MCP 세 가지 얼굴로 재사용**하며,
같은 on-disk `RunHistoryStore`를 공유하므로 MCP로 만든 데이터가 GUI 앱에도 그대로 보입니다.

---

## 4. 설치 및 사용법

### 전제 조건

- macOS 14 이상 (Apple Silicon 또는 Intel)
- Xcode
- Homebrew
- 시스템 설정 → 개인정보 보호에서 **화면 기록**, **손쉬운 사용** 권한 허용

> 리눅스 / 윈도우에서는 실행되지 않습니다. 코드 열람과 수정만 가능합니다.

### 길 A — 완성품 다운로드 (가장 쉬움)

1. https://github.com/awizemann/harness/releases/latest 에서 `Harness-v0.8.4-Universal.zip` 다운로드 (~12MB)
2. 압축 해제 후 `Harness.app` 실행
3. 첫 실행 마법사(FirstRunWizard)에서 안내대로 설정

### 길 B — 소스 빌드 (MCP 연동에 필요)

```bash
git submodule update --init --recursive   # appium/WebDriverAgent 벤더링
brew install xcodegen
xcodegen generate                          # Harness.xcodeproj 생성
open Harness.xcodeproj
```

- `Harness.xcodeproj`는 git에 없습니다. `project.yml`에서 **매번 생성**하므로,
  소스나 리소스를 바꾼 뒤에는 `xcodegen generate`를 다시 실행해야 합니다.
- 첫 실행 시 시뮬레이터 iOS 런타임에 맞춰 WebDriverAgent를 빌드합니다(1~2분).
  결과는 `~/Library/Application Support/Harness/wda-build/<iOS-version>/`에 캐시됩니다.

### MCP 서버 빌드

```bash
xcodebuild -project Harness.xcodeproj -scheme HarnessMCP -configuration Debug \
  -derivedDataPath ./.build/derived build
```

산출물: `./.build/derived/Build/Products/Debug/harness-mcp`
`.mcp.json`에 `harness` 서버로 이미 등록되어 있으므로, 빌드 후 MCP 클라이언트를 재시작하면 연결됩니다.

### 사용 흐름

```
1) Application 등록   — 웹 URL / iOS 프로젝트+스킴 / macOS .app 경로
2) Persona 작성       — 예: "60대, 스마트폰에 서툰 사용자"
3) Goal 입력          — 예: "장바구니에 담고 결제 직전까지 진행"
4) 실행               — 실시간으로 에이전트 동작 확인
5) 결과 확인          — 성공 여부 + 리플레이 + friction 리포트
```

---

## 5. 플러그인? 스킬? MCP?

| 구분 | 해당 여부 | 비고 |
| --- | :---: | --- |
| Claude Code 플러그인 | 아니오 | 플러그인 정의 파일 없음 |
| Claude Code 스킬 | 아니오 | `SKILL.md` 없음 |
| **MCP 서버** | **예** | `HarnessMCP/` |
| **독립 macOS 앱** | **예** | 사실상 본체 |
| **CLI 도구** | **예** | `HarnessCLI/` |

즉 **"macOS 앱 + MCP 서버"** 조합입니다.

### MCP 도구 목록 (요약)

**라이브러리 / 실행 관리**

| 도구 | 용도 |
| --- | --- |
| `list_personas` / `create_persona` | 테스트 사용자 프로필 |
| `list_applications` / `create_application` | 실행 대상 등록 |
| `list_actions` / `create_action` | 재사용 작업 프롬프트 |
| `list_action_chains` / `create_action_chain` | 다단계 연속 실행 |
| `stage_credential` / `list_credentials` / `delete_credential` | 로그인 정보 (비밀번호는 Keychain 전용) |
| `start_run` / `get_run_status` / `list_runs` | 자율 실행 시작 및 조회 |
| `get_run_result` / `get_step_screenshot` | 결과 + 요약 + **비용** / 스텝별 PNG |
| `list_agent_tools` | 플랫폼별 도구 introspection |

**스텝 단위 UI 세션 (LLM 루프 없음, API 키 불필요)**

| 도구 | 용도 |
| --- | --- |
| `start_ui_session` | 타겟 실행 및 세션 시작 (web / ios / macos) |
| `observe_ui` | 현재 화면 캡처 + 번호 마크 테이블 + `structuredContent` |
| `act_ui` | 동작 1회 수행 후 자동 재관찰 |
| `end_ui_session` | 세션 종료 (idempotent) |
| `list_ui_sessions` | 열린 세션 목록 |
| `export_ui_session_state` | (웹 전용) 쿠키 + localStorage 내보내기 — **민감 정보** |

---

## 6. API 토큰이 필요한가?

경로에 따라 다릅니다.

| 경로 | 토큰 | 설명 |
| --- | :---: | --- |
| 클라우드 LLM | **필요** | Anthropic / OpenAI / Google. 실행마다 비용 발생 |
| 로컬 Ollama | 불필요 | 무료, 오프라인, 프라이빗. 대신 느리고 품질이 낮음 |
| **MCP 스텝 세션** | **불필요** | Harness가 자체 LLM을 돌리지 않음. 판단은 외부 클라이언트가 함 |

- 키는 **macOS Keychain에만** 저장됩니다 (예: `com.harness.anthropic`). 디스크나 로그에 남지 않습니다.
- 로컬 모델 준비: `brew install ollama` → `ollama pull qwen3-vl:8b`
- MCP 스텝 세션은 README에 명시적으로 *"No API key is ever required on this path"* 라고 되어 있습니다.

---

## 7. GitHub에서의 위치 (2026-09 확인 기준)

```
Stars   347
Forks    22
생성일   2026-05-03
언어     Swift
라이선스 MIT
토픽     ai-agents, anthropic, claude, developer-tools, ios-testing,
        macos-testing, swift6, swiftui, user-testing, ux-testing, web-testing
```

"초대형 프로젝트"는 아니지만, 4개월 만의 수치로는 준수한 편입니다. 주목받은 이유:

1. **틈새 공략** — iOS 시뮬레이터 + macOS 앱 + 웹을 하나로 묶은 도구는 드묾. 특히 macOS 앱 자동화는 희귀
2. **포지셔닝** — "통과/실패"가 아니라 **"사용자가 어디서 헤맸는가"**. QA가 아닌 UX 리서치 영역
3. **완성도** — 테스트 530개 통과, 위키 33페이지, 표준 문서 14개, 릴리즈 13회
4. **타이밍** — MCP 생태계 확산기에 맞춰 붙임
5. **릴리즈 노트 품질** — 예: *"type()이 no-op을 성공으로 보고하던 것을 고침"* 처럼 솔직하고 읽히는 문체

---

## 8. 로컬 에이전트를 만들 때 배울 점

이 저장소의 가장 큰 가치는 **"쓰는 것"보다 "배우는 것"** 에 있습니다.

### 8.1 Set-of-Mark (SoM)

좌표로 클릭을 지시하면 자주 실패합니다. 대신 화면에 번호 배지를 그려주고
에이전트는 `tap_mark(3)`처럼 **id로 지목**합니다. 배지는 에이전트에게만 보이며 디스크에 저장되지 않습니다.

### 8.2 접근 가능한 이름 해석 순서 (웹)

```
aria-label → labelledby → <label> → placeholder → title → value
→ text → img-alt → svg-title → text-content → glyph → testid → name → synthesized
```

- 어떤 규칙이 이겼는지를 **`label_source`** 로 함께 보고합니다.
- `placeholder` / `value`는 샘플 데이터이므로 **약한 근거**로 취급해야 합니다.
- 이름을 못 찾은 컨트롤은 `unlabelled <role>` + `label_source: "synthesized"`로 표시되며,
  README는 이를 **페이지 접근성 버그로 취급하라**고 명시합니다.

### 8.3 선언되지 않은 모달 감지

React 팝업은 `role="dialog"`를 안 다는 경우가 많습니다. 다음 조건을 **모두** 만족하면 모달로 간주합니다.

- 선언된 모달이 없음 (선언되어 있으면 그걸 우선)
- `position: fixed` 이면서 양축 90% 이상 덮음
- **딤 처리** — 배경색 알파가 0과 1 사이 (`rgba()`, `oklch(… / .5)`, `color-mix()` 모두 인식)
- 살아있는 컨트롤 + **중앙에 위치한 콘텐츠 박스**(40×40 이상, 뷰포트 75% 이하, 불투명 배경)를 포함

불확실하면 "모달 아님"으로 degrade — 틀린 판정보다 낫다는 원칙입니다.

### 8.4 언제 스크린샷을 찍을 것인가 (settle)

동작을 **디스패치하기 전에** 관찰을 무장해두고, 페이지의 대기 작업(`setTimeout` ≤ 2s, in-flight `fetch`/XHR)을
소진시킵니다. 하한 250ms, 상한 3초. 이렇게 하지 않으면 **변경 전 프레임**을 찍어 에이전트가 혼란에 빠집니다.

### 8.5 구조화된 관찰 (`structuredContent`)

텍스트 마크 테이블이 버리는 **기하 정보(rect)** 를 함께 반환합니다.

- 좌표계는 **point space** (웹은 CSS 픽셀). 스크린샷 픽셀 공간과 다릅니다.
- `page_text`는 최대 20,000자, 모달 규칙과 동일하게 스코프됩니다.
- `frame_url`은 **의도적으로 검열** — scheme/host/port/path만, 쿼리스트링과 fragment는 폐기.

### 8.6 MCP 서버 구현 참고

```
MCPProtocol.swift   JSON-RPC 프로토콜
MCPServer.swift     stdio 루프
ToolRegistry.swift  도구 등록
ToolHandlers.swift  처리 로직
UISessionTools.swift 세션 도구
```

`outputSchema` 기반 구조화 응답까지 구현되어 있어 실전 예제로 적합합니다.

### 8.7 비밀 정보 취급

- 비밀번호는 Keychain에만 저장
- `steps.jsonl` 등 로그에 값이 남지 않음
- URL에서 토큰이 담길 수 있는 부분 제거
- 에러 바디도 redact

### 8.8 프롬프트 원본

`docs/PROMPTS/`에 시스템 프롬프트, 페르소나 기본값, friction 어휘집이 그대로 들어 있습니다.

---

## 9. React / PHP로 다시 만들 수 있을까?

| 기능 | React/Node로 가능? | 비고 |
| --- | :---: | --- |
| 웹 테스트 | **가능** | Playwright가 WKWebView보다 유리 |
| SoM 번호 배지 | 가능 | DOM 읽고 오버레이 그리면 됨 |
| 스크린샷 | 가능 | Playwright `screenshot()` |
| 리포트 대시보드 | **더 유리** | SwiftUI보다 쉬움 |
| MCP 서버 | 가능 | 공식 TypeScript SDK |
| iOS 시뮬레이터 | **불가** | `xcodebuild` / `simctl` = macOS 전용 |
| macOS 앱 | **불가** | 접근성 API = macOS 전용 |

### 제안 구조

```
React + Next.js         프론트엔드 (목표 입력 / 실시간 뷰 / 리포트)
        │ WebSocket
Node.js                 에이전트 엔진
        ├─ Playwright   브라우저 구동
        ├─ SoM 주입     번호 배지 JS
        ├─ LLM 호출     Claude / GPT / Ollama
        └─ 에이전트 루프 관찰 → 판단 → 행동
        │
PHP(Laravel) 또는 Node  회원 / 결제 / 기록 / 관리자
        │
PostgreSQL + 오브젝트 스토리지(스크린샷)
```

- **PHP 단독은 비권장** — 브라우저 자동화 생태계가 Node 중심입니다.
  PHP는 백엔드(회원/결제/관리자), Node는 엔진으로 나누는 편이 현실적입니다.

### 포팅 시 얻는 것

| Swift 원본 | React/Node 버전 |
| --- | --- |
| macOS 필수 | 리눅스 서버 포함 어디서나 |
| WebKit만 | Chromium / Firefox / WebKit |
| 1인 1대 | 서버에서 병렬 실행 |
| 앱 다운로드 배포 | URL 하나로 배포 |
| Swift 개발자 희소 | 인력 확보 용이 |

### 대략적 일정

```
1주차   Playwright로 브라우저 구동 + 스크린샷
2주차   SoM 번호 배지 JS
3주차   LLM에 스크린샷 전달 → 다음 행동 결정
4주차   에이전트 루프 완성
5~6주차 React 대시보드 + 리플레이
7~8주차 friction 리포트 + 회원/결제
```

---

## 10. 수익화 검토

> 아래 가격과 시장 판단은 **검증된 수치가 아니라 가설**입니다. 실제 고객 인터뷰로 검증이 필요합니다.

### 10.1 전제

MIT 코드는 해자가 아닙니다. 누구나 복사할 수 있습니다.
실제 해자는 **특정 고객의 특정 고통에 대한 이해 + 그 고객에게 닿는 경로 + 축적된 데이터**입니다.

### 10.2 수익 모델 유형

| 유형 | 선투자 | 수익 속도 | 확장성 |
| --- | --- | --- | --- |
| 서비스(컨설팅) | 낮음 | 빠름 | 낮음 |
| SaaS(구독) | 높음 | 느림 | 높음 |
| 툴 1회 판매 | 중간 | 중간 | 중간 |
| 교육 콘텐츠 | 낮음 | 중간 | 중간 |
| 오픈코어 | 높음 | 느림 | 높음 |

소규모라면 **서비스로 시작 → 반복 구간을 SaaS로 자동화**가 정석입니다.

### 10.3 아이디어별 정리

| 아이디어 | 핵심 | 첫 매출 | 확장성 | 주요 리스크 |
| --- | --- | --- | --- | --- |
| **웹 접근성 자동 진단** | 로그인 뒤 화면까지 검사. `synthesized` 라벨이 곧 접근성 결함 신호 | 3~6개월 | 높음 | 오탐 시 신뢰 상실, 법적 보증 불가 명시 필요 |
| **커머스 전환율 진단** | ROI 설명이 즉시 됨 ("전환율 1% = 월 100만원") | 2~4개월 | 높음 | 실결제 불가, 캡차, 개선 효과 입증 필요 |
| **UX 진단 컨설팅** | 도구는 내부 무기, 결과물(리포트)을 판매 | **2~4주** | 낮음 | 시간 판매라 확장 안 됨 |
| **CI/CD 통합** | PR마다 UX 회귀 체크 | 4~8개월 | 높음 | 느리거나 불안정하면 즉시 무시당함 |
| **교육 콘텐츠** | SoM / 라벨 우선순위 / 모달 감지를 교재화 | 1~2개월 | 중간 | 수익 규모 작음, 마케팅 채널 필요 |
| **리뉴얼 전후 검증** | 같은 목표로 구/신 버전 비교 | 1~3개월 | 낮음 | 일감이 간헐적 |
| **macOS/iOS 전문 QA** | 경쟁자 거의 없음 (원본 그대로 활용) | 3~6개월 | 중간 | 시장 규모 작음 |

### 10.4 접근성 진단이 유망한 이유

코드에서 발견한 지점입니다. 웹 프로브가 이름을 찾지 못한 컨트롤을
`unlabelled <role>` + `label_source: "synthesized"` 로 보고하고,
README가 이를 **접근성 버그로 취급하라**고 명시합니다.
즉 UX 테스트의 부산물로 **접근성 결함 탐지기**가 따라옵니다.

기존 정적 스캐너(axe, Lighthouse) 대비 차별점:

| | 기존 도구 | Harness 기반 |
| --- | --- | --- |
| 방식 | 정적 HTML 스캔 | 실제로 조작해봄 |
| 로그인 뒤 화면 | 접근 어려움 | `session_state` 주입으로 접근 |
| 모달/팝업 | 열지 않음 | 열어서 검사 |
| 다단계 폼 | 첫 페이지만 | 끝까지 진행 |

> 한국의 웹 접근성 관련 법령·인증 요건은 **직접 최신 정보를 확인**해야 합니다.

### 10.5 원가 구조

SaaS 실패의 흔한 원인은 LLM 원가 미계산입니다.

```
1회 실행 = 20~40 스텝
1 스텝  = 스크린샷(이미지 토큰) + 텍스트 + 응답
```

**추측하지 말고 측정하세요.** `get_run_result`가 **cost를 직접 반환**합니다.

```
1) 실제 사이트로 10회 실행
2) cost 평균 산출
3) × 3 (재시도/실패/긴 플로우 여유분)
4) = 1회 실제 원가
5) 판매가 >= 원가 × 5
```

원가 절감 수단:

- 프롬프트 캐싱 (이미 구현되어 있음)
- 이미지 다운스케일 (point size로 축소 전송)
- 마크 테이블만으로 판단 가능하면 이미지 생략
- 작은 모델 우선, 막히면 상위 모델로 승급
- 단순 반복은 로컬 Ollama로

**무제한 요금제는 피하세요.** 헤비 유저 1명이 마진을 전부 태웁니다.
"월 정액 + 실행 횟수 한도 + 초과분 종량제" 구조를 권장합니다.

### 10.6 라이선스 및 법적 체크

MIT 라이선스에서 **가능한 것**: 상업적 판매, 수정, 비공개 전환, 재배포
MIT 라이선스의 **의무**: 저작권 표시 유지, 라이선스 전문 포함

```
배포물에 포함할 것:
  MIT License
  Copyright (c) 2026 Alan Wizemann
  (전문 그대로)
```

권장 매너:

1. 완전히 다른 이름으로 브랜딩 ("Harness" 상표 혼동 회피)
2. `Inspired by / built on Harness by Alan Wizemann` 명시
3. 개선 사항은 upstream에 PR로 환원
4. 가능하면 원작자에게 사전 고지

추가로 챙길 것:

- **개인정보** — 고객 사이트 스크린샷에 개인정보가 찍힙니다. 보관 정책 / 암호화 / 파기 기한 필요
- **이용약관** — "AI 판단은 참고용이며 법적 보증이 아님" 명시
- **크롤링 동의** — 고객 본인 소유 자산만 검사한다는 확인
- **통신판매업 신고** 등 전자상거래 요건

### 10.7 권장 진행 순서

```
[1~2개월]              [3~5개월]               [6개월~]
컨설팅 3건 + 콘텐츠  →  반복 작업 자동화       →  SaaS 정식 출시
현금 확보 / 문제 학습     베타 10팀 / 제품 검증     유료 전환 / 확장
```

고객이 무엇을 원하는지는 **만들면서가 아니라 팔면서** 알게 됩니다.

### 10.8 첫 30일 플랜

```
1주차  Playwright + SoM 최소 동작 / 사이트 3곳 시험 / 1장짜리 리포트 템플릿
2주차  지인 5곳에 무료 진단 제공 (조건: 피드백 30분)
3주차  "돈 내고 쓸 의향? 얼마?" 직접 질문
4주차  첫 유료 전환 시도 + 블로그/영상 1편
```

### 10.9 검증 신호

**긍정 신호**

- "이거 다른 사람한테 공유해도 되나요?" (내부 공유 = 가치 인정)
- "언제 또 해주세요?" (반복 수요)
- 먼저 가격을 물어봄
- 지적한 항목을 실제로 수정함

**부정 신호**

- "신기하네요" 하고 끝
- "우리도 알고 있었어요"
- 결정권자를 못 찾음
- 무료인데도 사용하지 않음

> "좋네요"는 신호가 아니고, "언제 시작해요?"가 신호입니다.

---

## 11. 주의사항 요약

1. 이 저장소는 **Alan Wizemann**의 프로젝트를 fork한 것입니다. 원본은 https://github.com/awizemann/harness
2. **macOS 14 이상 필수** — 리눅스/윈도우에서는 빌드·실행 불가
3. `.mcp.json`의 `harness` 서버는 **바이너리를 빌드해야** 동작합니다
4. `memophant` MCP 서버는 별도 macOS 앱이 필요합니다
5. `Harness.xcodeproj`는 git에 없으므로 `xcodegen generate`가 필요합니다
6. README 내부의 다운로드·위키 링크는 모두 **원본 저장소**를 가리킵니다

---

*이 문서는 저장소 내용을 직접 확인해 작성한 한국어 정리본입니다. 코드 동작을 변경하지 않습니다.*
