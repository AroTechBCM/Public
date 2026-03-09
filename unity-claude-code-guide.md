# 🎮 Unity 개발자를 위한 Claude Code 가이드
## AI 코딩 도구를 처음 도입하는 경력 개발자용

> **대상**: Unity C# 개발자 (경력 5년 이상, AI 코딩 도구 미경험)  
> **프로젝트**: 야구매니저  
> **버전**: 1.0 (2026.03)  
> **작성**: KDM

---

## 목차

1. [왜 AI 코딩 도구인가](#1-왜-ai-코딩-도구인가)
2. [Claude Code 개요](#2-claude-code-개요)
3. [설치 및 세팅](#3-설치-및-세팅)
4. [CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기](#4-claudemd--ai에게-프로젝트-컨텍스트-주입하기)
5. [Unity 개발 워크플로우](#5-unity-개발-워크플로우)
6. [실전 활용 예시](#6-실전-활용-예시)
7. [Unity MCP — AI가 에디터를 직접 조작하게 하기](#7-unity-mcp--ai가-에디터를-직접-조작하게-하기)
8. [효과적인 프롬프트 작성법](#8-효과적인-프롬프트-작성법)
9. [주의사항과 한계](#9-주의사항과-한계)
10. [레퍼런스](#10-레퍼런스)

---

## 1. 왜 AI 코딩 도구인가

### 솔직한 현실 인식

여러분은 이미 Unity에서 수백 개의 MonoBehaviour를 작성하고, ScriptableObject 아키텍처를 설계하고, Addressable을 다뤄본 경력자입니다. AI가 여러분의 자리를 대체하는 것이 아닙니다.

AI 코딩 도구가 바꾸는 것은 **작업의 밀도**입니다:

| 작업 유형 | 기존 방식 | Claude Code 활용 |
|---|---|---|
| 보일러플레이트 코드 | 직접 타이핑 또는 스니펫 복사 | 컨텍스트 기반 자동 생성 |
| 리팩토링 | 수동 검색-치환, 파일별 수정 | 프로젝트 전체를 보고 일괄 리팩토링 |
| 디버깅 | 콘솔 로그 → 추론 → 수정 반복 | 에러 메시지 + 코드 컨텍스트로 원인 분석 |
| 새 시스템 설계 | 문서 조사 → 프로토타입 → 반복 | 기존 아키텍처 참조하여 초안 생성 |
| 테스트 코드 | 귀찮아서 후순위 | 기존 코드 기반 테스트 자동 생성 |
| 문서화 | XML 주석 수작업 | 코드 분석 후 주석/문서 일괄 생성 |

### 어떤 것이 아닌지

- ❌ "AI가 게임을 만들어줘" → 동작하지 않습니다
- ❌ Copilot 스타일 자동완성 → 그것보다 훨씬 큰 범위입니다
- ❌ 프로그래밍을 몰라도 되는 도구 → 오히려 경력자일수록 더 잘 활용합니다

### 어떤 것인지

- ✅ 프로젝트 폴더 전체를 컨텍스트로 이해하는 **코딩 에이전트**
- ✅ 기존 아키텍처, 네이밍 컨벤션, 의존성을 파악한 상태에서 코드를 생성
- ✅ 여러 파일을 동시에 수정하고, diff를 보여주고, 승인을 받는 방식
- ✅ 자연어로 지시하되, 기술적 맥락을 제공할수록 품질이 올라감

---

## 2. Claude Code 개요

### Claude Code란

Anthropic이 만든 **에이전틱 코딩 도구**입니다. VS Code 확장으로 설치하며, 프로젝트 디렉토리 전체를 컨텍스트로 활용하여 코드를 읽고, 생성하고, 수정합니다.

일반적인 AI 채팅(Claude 웹, ChatGPT)과의 핵심 차이:

- **파일 시스템 접근**: 프로젝트 내 모든 `.cs`, `.asset`, `.prefab`, `.json` 파일을 직접 읽고 쓸 수 있음
- **CLAUDE.md**: 프로젝트 루트에 위치한 규칙 파일을 자동으로 읽어, 매번 컨텍스트를 설명할 필요 없음
- **diff 기반 수정**: 모든 파일 수정을 diff로 보여주고 Accept/Reject 선택 가능
- **대화 연속성**: 같은 세션 내에서 이전 지시와 수정 이력을 기억

### 비용

Claude Max 구독 또는 Anthropic API 키가 필요합니다. 회사 계정 정보는 팀장에게 문의하세요.  
세션 중 `/cost` 를 입력하면 현재까지의 토큰 사용량과 비용을 확인할 수 있습니다.

---

## 3. 설치 및 세팅

여러분은 개발자이므로 간결하게 갑니다.

### 3-1. VS Code + 확장 설치

1. VS Code가 없다면: [https://code.visualstudio.com](https://code.visualstudio.com)
2. VS Code에서 `Ctrl + Shift + X` → **"Claude Code"** 검색 → Publisher가 **Anthropic** 인 것 Install
3. 설치 후 왼쪽 사이드바에 ✱ (Spark) 아이콘 확인
4. 최초 클릭 시 Anthropic OAuth 로그인

> 💡 CLI도 함께 사용하고 싶다면 터미널에서:
> ```bash
> npm install -g @anthropic-ai/claude-code
> claude auth login
> ```
> VS Code 확장과 CLI는 동일한 백엔드를 사용합니다. 상황에 따라 혼용 가능합니다.

### 3-2. Unity 프로젝트 폴더 열기

VS Code에서 **Unity 프로젝트 루트** (Assets, Packages, ProjectSettings가 있는 폴더)를 엽니다.

```
File → Open Folder → D:\Projects\BaseballManager
```

Claude Code가 프로젝트 구조를 인식하려면 **반드시 프로젝트 루트를 열어야** 합니다.  
`Assets/Scripts` 같은 하위 폴더만 열면 전체 컨텍스트를 파악할 수 없습니다.

### 3-3. .gitignore 및 .claudeignore

Claude Code는 프로젝트 내 파일을 읽으므로, 불필요한 파일을 제외시켜야 합니다.  
프로젝트 루트에 `.claudeignore` 파일을 생성하세요:

```
# Unity 생성 파일 (읽을 필요 없음)
Library/
Temp/
Obj/
Logs/
UserSettings/
MemoryCaptures/
Recordings/

# 빌드 산출물
Build/
Builds/
AssetBundles/

# IDE 설정
.vs/
.idea/

# 대용량 에셋 (토큰 낭비 방지)
Assets/Plugins/
Assets/ThirdParty/
Assets/TextMesh Pro/
*.dll
*.so
*.bundle

# 바이너리 에셋
*.png
*.jpg
*.wav
*.mp3
*.fbx
*.anim
```

이렇게 하면 Claude가 실제 프로젝트 코드에만 집중하게 되어 **응답 품질이 올라가고 비용이 줄어듭니다**.

---

## 4. CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기

### CLAUDE.md가 하는 일

프로젝트 루트에 `CLAUDE.md`를 놓으면, Claude Code가 **모든 대화의 시작 시점에 이 파일을 자동으로 읽습니다**.  
즉, 매번 "우리 프로젝트는 Unity 2022 LTS를 사용하고, Zenject로 DI하고..." 같은 설명을 반복할 필요가 없습니다.

이것은 사실상 **AI에 대한 프로젝트 온보딩 문서**입니다.

### Unity 프로젝트용 CLAUDE.md 예시

```markdown
# 야구매니저 — Unity 프로젝트 규칙

## 프로젝트 정보
- 게임명: 야구매니저
- 엔진: Unity 2022.3 LTS (URP)
- 언어: C# (.NET Standard 2.1)
- 타겟 플랫폼: Android / iOS
- 최소 지원: Android 8.0, iOS 15

## 아키텍처
- DI: VContainer (Zenject 아님)
- 비동기 처리: UniTask (System.Threading.Tasks 사용하지 않음)
- Reactive: R3 (UniRx 아님)
- UI: UI Toolkit (UGUI 사용하지 않음)
- 네트워크: 자체 HTTP 클라이언트 (Assets/Scripts/Core/Network/)
- 씬 관리: Addressable Scene
- 상태 관리: ScriptableObject 기반 이벤트 시스템

## 코드 컨벤션
- 네이밍: Microsoft C# 코딩 규칙 준수
  - private 필드: _camelCase
  - public 프로퍼티: PascalCase
  - 메서드: PascalCase
  - 인터페이스: IPrefix
  - 상수: UPPER_SNAKE_CASE
- MonoBehaviour 상속은 최소화, 로직은 순수 C# 클래스로 분리
- 매직넘버 금지 → const 또는 ScriptableObject로 관리
- 새 스크립트 생성 시 namespace 필수: BaseballManager.[모듈명]
- 모든 public API에 XML 주석 필수

## 폴더 구조
Assets/
├── Scripts/
│   ├── Core/           ← 공용 시스템 (DI, Event, Network, Util)
│   ├── Game/           ← 게임 로직
│   │   ├── Player/     ← 선수 데이터, 스탯, 성장
│   │   ├── Match/      ← 경기 시뮬레이션
│   │   ├── Team/       ← 구단 운영
│   │   └── Economy/    ← 재화, 가챠, 상점
│   ├── UI/             ← UI Toolkit 관련
│   └── Editor/         ← 커스텀 에디터 도구
├── Resources/
├── AddressableAssets/
└── Tests/
    ├── EditMode/
    └── PlayMode/
```

## 핵심 규칙
- 새 코드 작성 시 기존 코드의 패턴을 먼저 파악하고 일관성 유지할 것
- async 메서드에서는 반드시 UniTask 사용 (Task, async void 금지)
- Awake/Start에서 의존성 직접 찾지 않음 → VContainer에서 주입
- ScriptableObject를 데이터 컨테이너 + 이벤트 채널로 활용
- 테스트 코드는 Assets/Tests/ 하위에 미러 구조로 배치
- 성능 민감 코드에는 반드시 Profiler 마커 추가

## AWS 백엔드 (참고용)
- API Gateway + Lambda (C#/.NET 8)
- DynamoDB (선수 데이터, 유저 데이터)
- S3 (에셋 번들)
- CloudFront (CDN)
- 엔드포인트 규칙: /api/v1/{resource}
- 인증: Cognito JWT

## 응답 규칙
- 모든 응답과 주석은 한국어로 작성할 것
- 코드 내 변수명/메서드명은 영어, 주석은 한국어
- 새 파일 생성 시 파일 상단에 요약 주석 포함
```

### CLAUDE.md 관리 팁

- Git에 커밋하여 팀원 모두가 같은 규칙을 공유하세요
- 프로젝트가 발전하면 CLAUDE.md도 업데이트하세요 (새 라이브러리 도입, 아키텍처 변경 등)
- 너무 길면 핵심만 남기고 상세 내용은 별도 마크다운으로 분리 가능
- 수정 후에는 Claude Code에서 **새 대화를 시작**해야 반영됩니다

---

## 5. Unity 개발 워크플로우

### 기본 작업 흐름

```
VS Code에서 Claude Code 패널 열기 (✱ 아이콘)
        ↓
자연어로 작업 지시
        ↓
Claude가 기존 코드를 분석하고 수정안 생성
        ↓
diff 화면에서 변경 사항 검토
        ↓
Accept / Reject 선택
        ↓
Unity 에디터로 돌아가서 컴파일 확인 및 테스트
```

### Unity + VS Code 병행 사용 패턴

Unity 에디터와 VS Code를 나란히 띄워두고 작업하는 것을 권장합니다.

- **VS Code (Claude Code)**: 코드 생성, 수정, 리팩토링
- **Unity 에디터**: 컴파일 결과 확인, Inspector 값 조정, 플레이 테스트

Claude Code가 `.cs` 파일을 수정하면 Unity 에디터가 자동으로 리컴파일합니다.  
컴파일 에러가 나면 콘솔 에러 메시지를 Claude에게 그대로 붙여넣으세요.

### @ 멘션으로 파일 참조하기

채팅 입력창에서 `@` 를 입력하면 프로젝트 내 파일 목록이 나타납니다.  
원하는 파일을 선택하면 Claude가 해당 파일의 내용을 직접 참조합니다.

```
@Assets/Scripts/Game/Match/MatchSimulator.cs 이 클래스의 SimulateAtBat 메서드를 
투수 구종별로 분기 처리하도록 리팩토링해줘. 기존 PitchType enum을 활용해줘.
```

이것은 Claude 웹에 코드를 복붙하는 것과 완전히 다릅니다.  
Claude Code는 `PitchType` enum이 어디에 정의되어 있는지, 누가 참조하는지까지 프로젝트를 탐색합니다.

---

## 6. 실전 활용 예시

### 예시 1: 새 시스템 스캐폴딩

```
선수 컨디션 시스템을 만들어줘.
요구사항:
- IPlayerCondition 인터페이스 정의
- PlayerConditionService 구현 (VContainer로 등록 가능하게)
- 피로도(0~100)는 경기 출장 시 증가, 휴식 시 감소
- 컨디션 상태: Best / Good / Normal / Bad / Worst (enum)
- 피로도 → 컨디션 매핑 공식 포함
- ScriptableObject로 수치 테이블 분리
- 기존 PlayerData와 연동 가능한 구조
- EditMode 테스트 포함
```

### 예시 2: 기존 코드 리팩토링

```
@Assets/Scripts/Game/Match/MatchSimulator.cs 를 분석해줘.

현재 문제:
- SimulateInning() 메서드가 300줄이 넘음
- 타격/투구/수비 로직이 한 메서드에 뒤섞여 있음

리팩토링 방향:
- 타격 판정을 BattingResolver 클래스로 분리
- 투구 판정을 PitchingResolver 클래스로 분리  
- 수비 판정을 FieldingResolver 클래스로 분리
- 각 Resolver는 VContainer로 주입 가능하게
- 기존 테스트가 깨지지 않도록 인터페이스 유지
```

### 예시 3: 버그 디버깅

```
Unity 콘솔에 아래 에러가 반복 발생해:

NullReferenceException: Object reference not set to an instance of an object
  at BaseballManager.Game.Match.LineupManager.GetCurrentBatter () 
  at BaseballManager.Game.Match.MatchSimulator.SimulateAtBat ()

@Assets/Scripts/Game/Match/LineupManager.cs
@Assets/Scripts/Game/Match/MatchSimulator.cs

원인을 분석하고 수정해줘. 
null 체크뿐 아니라 왜 null이 되는 근본 원인도 설명해줘.
```

### 예시 4: 테스트 코드 생성

```
@Assets/Scripts/Game/Economy/GachaService.cs 에 대한 
EditMode 테스트를 작성해줘.

테스트 케이스:
- 등급별 확률이 합산 100%인지 검증
- 천장(pity) 시스템이 정상 동작하는지 (N회 연속 꽝 후 SSR 보장)
- 시드값 고정 시 결과 재현성 검증
- 재화 부족 시 예외 처리 검증

NUnit + Unity Test Framework 사용, 
Assets/Tests/EditMode/Economy/ 에 생성해줘.
```

### 예시 5: ScriptableObject 데이터 생성

```
투수 스탯 데이터용 ScriptableObject를 만들어줘.

요구사항:
- PitcherStatsSO : ScriptableObject
- 필드: 선수명, 등급(enum), 구위, 제구, 스태미나, 멘탈, 구종 목록
- 커스텀 에디터: Inspector에서 스탯 합계와 등급 적정 범위를 실시간 표시
- CreateAssetMenu 어트리뷰트 포함
- 기존 @Assets/Scripts/Game/Player/PlayerData.cs 의 패턴을 따라줘
```

### 예시 6: 에디터 도구 제작

```
선수 데이터를 CSV에서 일괄 임포트하는 에디터 윈도우를 만들어줘.

요구사항:
- EditorWindow, 메뉴 경로: Tools/Baseball Manager/Import Player Data
- CSV 파일 드래그 앤 드롭 지원
- CSV 파싱 → PitcherStatsSO / BatterStatsSO 자동 생성
- 기존 에셋이 있으면 덮어쓸지 확인 다이얼로그
- 진행률 표시 (EditorUtility.DisplayProgressBar)
- 임포트 결과 로그 출력
```

### 예시 7: 성능 최적화

```
@Assets/Scripts/Game/Match/MatchSimulator.cs 의 성능을 분석해줘.

매 프레임 호출되는 부분에서:
- GC Alloc이 발생하는 구간 식별
- LINQ 사용을 for 루프로 대체
- string 연결을 StringBuilder로 변환
- 불필요한 GetComponent 호출 캐싱
- Profiler.BeginSample/EndSample 마커 추가

변경 전후 예상 영향도 설명해줘.
```

### 예시 8: AWS 연동 코드

```
선수 데이터를 DynamoDB에서 가져오는 API 클라이언트를 만들어줘.

요구사항:
- 엔드포인트: GET /api/v1/players/{playerId}
- 기존 @Assets/Scripts/Core/Network/ApiClient.cs 패턴을 따라
- UniTask 기반 비동기
- JSON 역직렬화 (Newtonsoft.Json)
- 에러 핸들링: 네트워크 오류, 타임아웃, 4xx/5xx 분기
- 재시도 로직 (최대 3회, exponential backoff)
- Cognito JWT 토큰 자동 갱신
```

---

## 7. Unity MCP — AI가 에디터를 직접 조작하게 하기

### Unity MCP란

MCP(Model Context Protocol)는 AI 도구와 외부 애플리케이션을 연결하는 프로토콜입니다.  
**Unity MCP**를 설치하면 Claude Code가 Unity 에디터에 직접 명령을 보낼 수 있습니다.

즉, "빨간 큐브를 만들어줘"라고 말하면 Claude가 `.cs` 파일을 생성하는 게 아니라,  
**Unity 에디터에서 직접 GameObject를 생성**합니다.

### 할 수 있는 것

- GameObject 생성/삭제/수정
- 컴포넌트 추가/제거/설정
- 머티리얼 생성 및 적용
- 씬 관리 (로드, 저장)
- 프리팹 생성
- 커스텀 에디터 메뉴 실행
- 콘솔 로그 읽기
- 테스트 실행
- 애니메이션, 카메라(Cinemachine), ProBuilder, UI, VFX 관리

### 설치 방법

**1단계: Unity 패키지 설치**

Unity 에디터에서:
```
Window → Package Manager → + → Add package from git URL...
```
URL 입력:
```
https://github.com/CoplayDev/unity-mcp.git
```

**2단계: Claude Code에 MCP 서버 등록**

VS Code 터미널 (`` Ctrl + ` `` 또는 상단 메뉴 **터미널 → 새 터미널**) 에서:
```bash
claude mcp add --scope user --transport stdio unity-mcp -- uvx --python ">=3.11" unity-mcp-server@latest
```

**3단계: 연결 확인**

Claude Code 채팅 패널에서:
```
Unity MCP로 현재 열려있는 씬의 GameObject 목록을 보여줘
```

정상적으로 응답이 오면 연결 성공입니다.

> ⚠️ Unity MCP는 아직 베타입니다. 프로덕션 씬에서 사용하기 전에 반드시 Git 커밋을 해두세요.

---

## 8. 효과적인 프롬프트 작성법

### 원칙: 코드 리뷰를 요청하듯 지시하라

여러분이 주니어 개발자에게 작업을 지시할 때를 떠올리세요.  
"로그인 만들어"가 아니라, 요구사항과 제약 조건을 명확히 전달하겠죠. 같은 원칙입니다.

### 나쁜 프롬프트 vs 좋은 프롬프트

**❌ 나쁨:**
```
가챠 시스템 만들어줘
```

**✅ 좋음:**
```
가챠 시스템을 만들어줘.

구조:
- IGachaService 인터페이스 + GachaService 구현
- VContainer에 등록 가능한 형태
- 확률 테이블은 GachaProbabilitySO (ScriptableObject)로 분리

로직:
- 등급별 가중치 랜덤 (N:60%, R:25%, SR:10%, SSR:4%, UR:1%)
- 천장 시스템: SSR 이상 미획득 80연차 시 SSR 보장
- 10연차 할인: 단가 300 다이아 → 10연차 2700 다이아

참고:
- @Assets/Scripts/Game/Economy/CurrencyService.cs 의 재화 차감 로직 활용
- @Assets/Scripts/Game/Player/PlayerData.cs 의 선수 데이터 구조 따르기

테스트:
- 확률 분포 검증 (10만회 시뮬레이션)
- 천장 정상 동작 검증
- 재화 부족 시 예외 검증
```

### 프롬프트 체크리스트

1. **무엇을** — 만들거나 수정할 대상
2. **어떤 패턴으로** — 기존 코드의 어떤 패턴을 따를지 (`@` 멘션)
3. **제약 조건** — 사용할/사용하지 않을 라이브러리, 성능 요구 등
4. **결과물 위치** — 어떤 폴더/네임스페이스에 생성할지
5. **테스트** — 테스트 코드 포함 여부와 검증 기준

### 점진적 작업 패턴

큰 시스템을 한 번에 만들라고 하면 품질이 떨어집니다.  
단계를 나눠서 진행하세요:

```
1단계: "경기 시뮬레이션 시스템의 인터페이스와 데이터 구조만 먼저 설계해줘"
2단계: "투구 판정 로직을 구현해줘"
3단계: "타격 판정 로직을 구현해줘, 투구 판정과 연동되게"
4단계: "수비 판정을 추가해줘"
5단계: "전체 시스템에 대한 테스트를 작성해줘"
```

---

## 9. 주의사항과 한계

### Git은 생명줄입니다

Claude Code는 파일을 직접 수정합니다. diff에서 Accept 후 잘못된 것을 발견하면 되돌리기 어렵습니다.

- **작업 전 반드시 커밋**하세요
- 대규모 리팩토링은 별도 브랜치에서 진행하세요
- Claude에게 작업을 시키기 전에 `git status`로 워킹 트리가 깨끗한지 확인하세요

### AI가 모르는 것들

Claude Code는 강력하지만 **Unity 런타임을 실행하지 않습니다**.

- Inspector에서 값이 어떻게 보이는지 모릅니다
- 특정 기기에서의 성능 특성을 모릅니다
- 에셋 번들의 실제 크기나 로딩 시간을 모릅니다
- Shader 컴파일 결과를 예측하지 못합니다
- 물리 시뮬레이션의 실제 체감을 판단하지 못합니다

수치 민감한 부분(물리 파라미터, 밸런스, 연출 타이밍)은 반드시 플레이 테스트로 검증하세요.

### AI 생성 코드의 검토 포인트

AI가 생성한 코드를 Accept 하기 전에 확인할 것:

1. **GC Alloc**: LINQ, string 연결, 람다 캡처 등 매 프레임 할당 발생 여부
2. **스레드 안전성**: UniTask와 메인 스레드 접근이 올바른지
3. **null 안전성**: ?. 와 ?? 가 적절히 사용되었는지
4. **네이밍 일관성**: 기존 코드와 네이밍 컨벤션이 맞는지
5. **의존성 방향**: 아키텍처 레이어를 역방향으로 참조하지 않는지
6. **에디터 전용 코드**: `#if UNITY_EDITOR` 가드가 필요한 곳에 빠지지 않았는지

### 과도한 의존을 경계하세요

AI 도구를 잘 활용하는 것과 AI 없이 코드를 작성할 수 없게 되는 것은 다릅니다.  
특히 핵심 게임 로직과 아키텍처 결정은 직접 설계하되, 구현의 속도를 높이는 데 활용하세요.

---

## 10. 레퍼런스

### 공식 문서

- Claude Code 공식 문서: [https://code.claude.com/docs](https://code.claude.com/docs)
- VS Code 확장: [https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)
- CLAUDE.md 작성 가이드: [https://code.claude.com/docs/en/claude-md](https://code.claude.com/docs/en/claude-md)

### Unity 관련

- Unity MCP (오픈소스): [https://github.com/CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp)
- Unity용 Claude Code 설정 예시: [https://github.com/nowsprinting/claude-code-settings-for-unity](https://github.com/nowsprinting/claude-code-settings-for-unity)
- Unity 개발용 Claude Code Skills: [https://github.com/The1Studio/theone-training-skills](https://github.com/The1Studio/theone-training-skills)

### 사내 자료

- 기획팀 Claude Code 매뉴얼: `vscode.md` (비개발자용)
- 프로젝트 CLAUDE.md: 프로젝트 루트에 위치

---

> 📝 이 가이드에 대한 질문이나 개선 사항은 CEO 미니에게 전달해주세요.  
> AI 도구 도입 과정에서 발견한 팁이나 문제가 있으면 팀 내 공유 부탁드립니다.
