# 🎮 언리얼 개발자를 위한 Claude Code 가이드
## AI 코딩 도구를 처음 도입하는 경력 개발자용

> **대상**: Unreal Engine C++ 개발자 (경력 5년 이상, AI 코딩 도구 미경험)  
> **프로젝트**: 야구매니저  
> **버전**: 1.0 (2026.03)  
> **작성**: CEO 미니

---

## 목차

1. [왜 AI 코딩 도구인가](#1-왜-ai-코딩-도구인가)
2. [Claude Code 개요](#2-claude-code-개요)
3. [설치 및 세팅](#3-설치-및-세팅)
4. [CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기](#4-claudemd--ai에게-프로젝트-컨텍스트-주입하기)
5. [Unreal 개발 워크플로우](#5-unreal-개발-워크플로우)
6. [실전 활용 예시](#6-실전-활용-예시)
7. [Unreal MCP — AI가 에디터를 직접 조작하게 하기](#7-unreal-mcp--ai가-에디터를-직접-조작하게-하기)
8. [효과적인 프롬프트 작성법](#8-효과적인-프롬프트-작성법)
9. [주의사항과 한계](#9-주의사항과-한계)
10. [레퍼런스](#10-레퍼런스)

---

## 1. 왜 AI 코딩 도구인가

### 경력 개발자에게 AI 도구가 의미하는 것

여러분은 이미 UObject 라이프사이클을 머릿속으로 그릴 수 있고, GAS로 어빌리티 시스템을 설계하고, Dedicated Server 리플리케이션 디버깅을 해본 분들입니다. AI가 여러분의 판단력을 대체하는 것이 아닙니다.

AI 코딩 도구가 바꾸는 것은 **작업의 밀도**입니다:

| 작업 유형 | 기존 방식 | Claude Code 활용 |
|---|---|---|
| 보일러플레이트 | UCLASS/UPROPERTY/UFUNCTION 매크로 수작업 | 설계 의도 설명 → .h/.cpp 쌍 자동 생성 |
| 리팩토링 | 수동 검색-치환, 파일별 수정 | 프로젝트 전체를 보고 헤더/구현 일괄 리팩토링 |
| 디버깅 | 로그 + 중단점 → 추론 → 수정 반복 | 크래시 콜스택 + 코드 컨텍스트로 원인 분석 |
| 새 시스템 설계 | 문서 조사 → 프로토타입 → 반복 | 기존 아키텍처 참조하여 초안 생성 |
| 리플리케이션 | 프로퍼티/RPC 설정 수작업 | 동기화 요구사항 설명 → 리플리케이션 코드 생성 |
| 에디터 도구 | Slate/Detail Customization 수작업 | 커스텀 에디터 기능 설명 → 에디터 모듈 생성 |

### 어떤 것인지

- ✅ 프로젝트 폴더 전체(`.h`, `.cpp`, `.uproject`, `.uplugin`, `Config/`)를 컨텍스트로 이해하는 **코딩 에이전트**
- ✅ 기존 클래스 계층, 네이밍 컨벤션, 모듈 구조를 파악한 상태에서 코드를 생성
- ✅ `.h`와 `.cpp`를 동시에 수정하고, diff를 보여주고, 승인을 받는 방식
- ✅ MCP를 통해 Unreal Editor를 직접 조작 가능 (액터 생성, 씬 조작, BP 편집 등)

---

## 2. Claude Code 개요

### Claude Code란

Anthropic이 만든 **에이전틱 코딩 도구**입니다. VS Code 확장 또는 CLI로 사용하며, 프로젝트 디렉토리 전체를 컨텍스트로 활용합니다.

핵심 특징:

- **파일 시스템 접근**: `.h`, `.cpp`, `.ini`, `.uproject` 등 모든 파일을 직접 읽고 쓸 수 있음
- **CLAUDE.md**: 프로젝트 루트의 규칙 파일을 자동으로 읽어 매번 컨텍스트 설명 불필요
- **diff 기반 수정**: `.h`와 `.cpp`를 동시에 수정해도 각각 diff를 보여주고 Accept/Reject 가능
- **MCP 연동**: Unreal MCP로 에디터를 직접 제어 가능
- **셸 실행**: `UnrealBuildTool`, `RunUAT` 등 빌드 명령을 터미널에서 직접 실행 가능 (사용자 승인 필요)

### 비용

Claude Max 구독 또는 Anthropic API 키가 필요합니다. 회사 계정 정보는 팀장에게 문의하세요.  
세션 중 `/cost`를 입력하면 현재까지의 토큰 사용량과 비용을 확인할 수 있습니다.

### Rider / Visual Studio 사용자에게

Claude Code 확장은 현재 **VS Code 전용**입니다.  
Rider나 Visual Studio를 메인 IDE로 사용하더라도, Claude Code 작업은 VS Code에서 진행하세요.  
코드 작성은 VS Code + Claude Code, 에디터 작업은 Unreal Editor, 디버깅은 Rider/VS를 병행하는 패턴을 권장합니다.

---

## 3. 설치 및 세팅

### 3-1. VS Code 확장 설치

1. VS Code에서 `Ctrl + Shift + X` → **"Claude Code"** 검색 → Publisher: **Anthropic** Install
2. 왼쪽 사이드바에 ✱ (Spark) 아이콘 확인
3. 최초 클릭 시 Anthropic OAuth 로그인

### 3-2. CLI 설치 (선택사항이지만 권장)

```bash
npm install -g @anthropic-ai/claude-code
claude auth login
```

CLI는 `--print` 모드로 스크립팅이 가능하고, Unreal 빌드 파이프라인과 연계할 때 유용합니다.

### 3-3. 프로젝트 폴더 열기

VS Code에서 **`.uproject` 파일이 있는 프로젝트 루트**를 엽니다.

```
File → Open Folder → D:\Projects\BaseballManager
```

> ⚠️ `Source/` 하위 폴더만 열면 안 됩니다.  
> Claude가 `.uproject`, `Config/`, `Plugins/` 등 전체 프로젝트 구조를 파악해야 합니다.

### 3-4. .claudeignore

Unreal 프로젝트는 바이너리와 생성 파일이 매우 많으므로, 반드시 제외 설정을 해야 합니다:

```
# Unreal 생성 디렉토리
Binaries/
Intermediate/
DerivedDataCache/
Saved/
.vs/
.idea/

# 빌드 산출물
Build/
Packages/

# 에셋 (바이너리, 토큰 낭비 방지)
Content/
*.uasset
*.umap

# 써드파티 플러그인 소스 (필요 시 개별 해제)
Plugins/ThirdParty/

# 기타 바이너리
*.dll
*.lib
*.pdb
*.exe
*.o
*.a
*.so
*.dylib
```

> 💡 `Content/`를 제외하는 이유: `.uasset`은 바이너리이므로 Claude가 읽을 수 없고, 토큰만 낭비합니다.  
> 에셋 구조를 알려주고 싶다면 CLAUDE.md에 텍스트로 기술하세요.

---

## 4. CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기

### CLAUDE.md가 하는 일

프로젝트 루트에 `CLAUDE.md`를 놓으면, Claude Code가 **모든 대화의 시작 시점에 이 파일을 자동으로 읽습니다**. 매번 "UE 5.4, GAS 사용, Dedicated Server..." 같은 설명을 반복할 필요가 없습니다.

이것은 사실상 **AI에 대한 프로젝트 온보딩 문서**입니다.

### 야구매니저 UE 프로젝트용 CLAUDE.md 예시

```markdown
# 야구매니저 — Unreal Engine 프로젝트 규칙

## 프로젝트 정보
- 게임명: 야구매니저
- 엔진: Unreal Engine 5.4
- 빌드 타겟: Android / iOS / Windows
- 네트워크: Dedicated Server (리플리케이션)
- 소스 컨트롤: Git + Git LFS

## 모듈 구조
Source/
├── BaseballManager/              ← 메인 게임 모듈
│   ├── BaseballManager.Build.cs
│   ├── Core/                     ← 게임 프레임워크, GameMode, GameState
│   ├── Player/                   ← 선수 데이터, 스탯, 성장
│   ├── Match/                    ← 경기 시뮬레이션
│   ├── Team/                     ← 구단 운영, 로스터
│   ├── Economy/                  ← 재화, 가챠, 상점
│   ├── UI/                       ← UMG 위젯, 뷰모델
│   └── Network/                  ← 리플리케이션, RPC 래퍼
│
├── BaseballManagerEditor/        ← 에디터 전용 모듈
│   ├── BaseballManagerEditor.Build.cs
│   └── Tools/                    ← 커스텀 에디터 도구
│
└── BaseballManager.Target.cs
    BaseballManagerEditor.Target.cs
    BaseballManagerServer.Target.cs
```

## 아키텍처
- 게임플레이 프레임워크: GameplayAbilitySystem (GAS)
- 상태 관리: GameState/PlayerState를 통한 리플리케이션
- 데이터: DataTable + DataAsset (선수/스킬/아이템 데이터)
- UI: UMG + MVVM 패턴 (UMVVMViewModelBase 사용)
- 입력: Enhanced Input System
- 애니메이션: Animation Blueprint + Linked Anim Layers
- 네트워크: Dedicated Server, Push Model 리플리케이션 사용
- 비동기: 자체 AsyncTask 시스템 (GameThread 보장)

## C++ 코딩 컨벤션
- Unreal 코딩 표준 준수 (Epic 공식 가이드라인)
  - 클래스 접두사: A (Actor), U (UObject), F (구조체/순수C++), E (Enum), I (Interface), T (템플릿)
  - bool 변수: b 접두사 (bIsAlive, bCanMove)
  - UPROPERTY/UFUNCTION 매크로 필수 (Blueprint 노출 시)
  - 포인터: TObjectPtr<> 사용 (raw pointer 지양)
  - 컨테이너: TArray, TMap, TSet (STL 컨테이너 사용 금지)
- 헤더 가드: #pragma once
- 전방 선언 적극 활용 (헤더에서 #include 최소화)
- GENERATED_BODY() 위치: public 첫 줄
- 한 파일당 하나의 UCLASS/USTRUCT

## 리플리케이션 규칙
- 서버 권한 모델: 모든 게임 로직은 서버에서 실행
- 클라이언트는 입력 전송 + 결과 표시만 담당
- UPROPERTY(Replicated): 단방향 동기화 (서버→클라이언트)
- UPROPERTY(ReplicatedUsing=OnRep_*): 변경 시 콜백
- Server RPC: 클라이언트 → 서버 요청 (UFUNCTION(Server, Reliable))
- Client RPC: 서버 → 특정 클라이언트 (UFUNCTION(Client, Reliable))
- Multicast RPC: 서버 → 모든 클라이언트 (연출/이펙트용)
- Push Model: MARK_PROPERTY_DIRTY_FROM_NAME() 사용

## GAS (Gameplay Ability System) 규칙
- AbilitySystemComponent: PlayerState에 부착
- GameplayAbility: 투구/타격/수비 등 주요 게임플레이 액션
- GameplayEffect: 스탯 변경, 버프/디버프
- AttributeSet: 선수 스탯 (UBMAttributeSet_Pitcher, UBMAttributeSet_Batter)
- GameplayTag: BM.Ability.*, BM.Status.*, BM.Stat.* 형식
- GameplayEvent: 경기 이벤트 트리거용

## 빌드 / 모듈 규칙
- PublicDependencyModuleNames: 최소한으로 유지
- PrivateDependencyModuleNames: 구현부에서만 필요한 모듈
- IWYU (Include What You Use) 준수
- 순환 의존성 금지
- Editor 모듈은 런타임 모듈에서 참조 금지

## 테스트
- Automation Testing Framework 사용
- 테스트 위치: Source/BaseballManager/Tests/
- 핵심 시뮬레이션 로직은 자동화 테스트 필수
- 리플리케이션 테스트: Gauntlet 프레임워크

## 응답 규칙
- 모든 응답, 주석은 한국어
- 코드 내 변수명/함수명은 영어 (Unreal 컨벤션), 주석은 한국어
- 새 .h/.cpp 쌍 생성 시 파일 상단에 저작권 + 요약 주석
- UCLASS/UPROPERTY/UFUNCTION에 한국어 주석 포함
```

### CLAUDE.md 관리 팁

- Git에 커밋하여 팀원 모두가 동일한 규칙을 공유하세요
- GAS 태그 체계, 리플리케이션 규칙 등 **팀 내 암묵적 규칙**을 문서화해두면 AI가 일관된 코드를 생성합니다
- 수정 후에는 Claude Code에서 **새 대화를 시작**해야 반영됩니다

---

## 5. Unreal 개발 워크플로우

### 기본 작업 흐름

```
VS Code에서 Claude Code 패널 열기 (✱ 아이콘)
        ↓
자연어로 작업 지시 (+ 관련 파일 @멘션)
        ↓
Claude가 기존 .h/.cpp를 분석하고 수정안 생성
        ↓
diff 화면에서 변경 사항 검토 (.h와 .cpp 각각)
        ↓
Accept / Reject 선택
        ↓
Unreal Editor에서 컴파일 (또는 Hot Reload / Live Coding)
        ↓
에디터에서 플레이 테스트
```

### 3개 도구 병행 패턴

| 도구 | 용도 |
|---|---|
| **VS Code (Claude Code)** | 코드 생성, 수정, 리팩토링, .h/.cpp 동시 편집 |
| **Unreal Editor** | 컴파일, Blueprint 작업, 레벨 편집, 플레이 테스트 |
| **Rider / Visual Studio** | 디버깅, 브레이크포인트, 심볼 탐색 |

Claude Code가 `.h`/`.cpp` 파일을 수정하면, Unreal Editor에서 **Live Coding** (Ctrl+Alt+F11) 또는 에디터 재시작으로 컴파일을 확인하세요.

### @ 멘션으로 파일 참조하기

채팅 입력창에서 `@`를 입력하면 프로젝트 내 파일 목록이 나타납니다.

```
@Source/BaseballManager/Match/BMMatchSimulator.h 와
@Source/BaseballManager/Match/BMMatchSimulator.cpp 를 참고해서
이닝 시뮬레이션에서 투수 교체 로직을 추가해줘.
기존 PitcherState와 BullpenManager를 활용해줘.
```

Claude Code는 `BMMatchSimulator`의 클래스 계층, 의존하는 다른 클래스, 리플리케이션 설정까지 프로젝트를 탐색합니다.

---

## 6. 실전 활용 예시

### 예시 1: 새 게임플레이 시스템 스캐폴딩

```
선수 컨디션 시스템을 만들어줘.

요구사항:
- UBMConditionComponent : UActorComponent
  - 피로도(Fatigue, 0~100), 컨디션 상태(EBMConditionState enum)
  - 서버에서만 갱신, 클라이언트에 리플리케이션
  - UPROPERTY(ReplicatedUsing=OnRep_ConditionState)
  
- GAS 연동:
  - UBMGameplayEffect_FatigueAccumulate: 경기 출전 시 피로도 증가
  - UBMGameplayEffect_FatigueRecovery: 휴식 시 피로도 감소
  - 피로도가 AttributeSet의 Fatigue 어트리뷰트에 매핑

- 데이터:
  - FBMConditionConfig: DataAsset으로 수치 테이블 분리
  - 피로도 → 컨디션 매핑 커브 (UCurveFloat 참조)

기존 @Source/BaseballManager/Player/BMPlayerState.h 의 컴포넌트 부착 패턴과
@Source/BaseballManager/Match/BMMatchGameState.h 의 리플리케이션 패턴을 따라줘.
.h와 .cpp 쌍으로 만들고, Automation 테스트도 포함해줘.
```

### 예시 2: 리팩토링

```
@Source/BaseballManager/Match/BMMatchSimulator.cpp 를 분석해줘.

현재 문제:
- SimulateInning() 함수가 500줄이 넘음
- 타격/투구/수비/주루 로직이 한 함수에 뒤섞여 있음

리팩토링 방향:
- FBMBattingResolver 구조체로 타격 판정 분리
- FBMPitchingResolver 구조체로 투구 판정 분리
- FBMFieldingResolver 구조체로 수비 판정 분리
- FBMBaseRunningResolver 구조체로 주루 판정 분리
- 각 Resolver는 순수 C++ (UObject 아님, GC 불필요)
- SimulateInning()은 Resolver들을 조합하는 오케스트레이터 역할만

기존 테스트가 깨지지 않도록 인터페이스 유지하고,
.h와 .cpp 모두 수정해줘.
```

### 예시 3: 크래시 디버깅

```
QA에서 아래 크래시 콜스택이 반복 보고돼:

Assertion failed: IsValid()
UnrealEngine!UObject::GetFName()
BaseballManager!UBMLineupManager::GetCurrentBatter() [BMLineupManager.cpp:87]
BaseballManager!ABMMatchSimulator::SimulateAtBat() [BMMatchSimulator.cpp:234]
BaseballManager!ABMMatchSimulator::Tick() [BMMatchSimulator.cpp:156]

재현 조건: 9회 말 대타 투입 직후

@Source/BaseballManager/Match/BMLineupManager.h
@Source/BaseballManager/Match/BMLineupManager.cpp
@Source/BaseballManager/Match/BMMatchSimulator.cpp

원인을 분석하고 수정해줘.
GC에 의한 댕글링 포인터 가능성도 확인하고,
TObjectPtr로 전환이 필요한 부분이 있으면 함께 수정해줘.
```

### 예시 4: GAS 어빌리티 구현

```
투수의 "특수 구종 투구" 어빌리티를 GAS로 만들어줘.

요구사항:
- UBMGameplayAbility_SpecialPitch : UGameplayAbility
- 활성화 조건: GameplayTag BM.Status.OnMound 보유, 스태미나 > 20
- 비용: 스태미나 15 소모 (GameplayEffect로)
- 쿨다운: 3이닝 (커스텀 쿨다운, 실시간 아님 이닝 기반)
- 효과: 타자의 컨택 스탯에 디버프 적용 (2타석 유지)
- 구종별 서브클래스: Slider, Curveball, Changeup (데이터 드리븐)
- 서버 실행, 클라이언트에는 결과만 Multicast RPC로 전달

기존 @Source/BaseballManager/Match/Abilities/ 폴더의 
어빌리티 패턴을 참고해줘.
GameplayTag 등록도 포함해줘.
```

### 예시 5: 리플리케이션 구현

```
실시간 경기 스코어보드를 리플리케이션으로 만들어줘.

요구사항:
- ABMMatchGameState에 스코어보드 데이터 추가
- FBMScoreboardData (USTRUCT): 이닝별 득점, 안타, 에러 
- TArray<FBMInningScore>를 FFastArraySerializer로 최적화
- Push Model 사용: 변경된 이닝 데이터만 전송
- OnRep 콜백에서 UI 위젯에 이벤트 브로드캐스트
- 서버에서만 수정, 클라이언트는 읽기 전용

기존 @Source/BaseballManager/Core/BMMatchGameState.h 와
@Source/BaseballManager/Network/ 폴더의 리플리케이션 패턴을 따라줘.
GetLifetimeReplicatedProps, DOREPLIFETIME 등 빠짐없이 포함해줘.
```

### 예시 6: 커스텀 에디터 도구

```
선수 데이터를 CSV에서 일괄 임포트하는 에디터 유틸리티를 만들어줘.

요구사항:
- UBMPlayerDataImporter : UEditorUtilityWidget (또는 Slate 윈도우)
- 메뉴 등록: Tools > Baseball Manager > Import Player Data
- CSV 파일 선택 다이얼로그
- CSV 파싱 → UBMPitcherDataAsset / UBMBatterDataAsset 자동 생성
- 기존 에셋이 있으면 덮어쓸지 확인 다이얼로그
- 진행률 표시 (FScopedSlowTask)
- 임포트 결과 로그

BaseballManagerEditor 모듈에 배치하고,
.Build.cs에 필요한 모듈 의존성도 추가해줘.
```

### 예시 7: 성능 최적화

```
@Source/BaseballManager/Match/BMMatchSimulator.cpp 의 Tick 성능을 분석해줘.

확인할 것:
- 매 Tick 불필요한 TArray 할당이 있는지
- FName 비교 대신 GameplayTag 비교를 사용하고 있는지
- FindComponent / GetAllActorsOfClass 같은 매 프레임 탐색이 있는지
- FString::Printf 대신 FString::Format 또는 직접 조합이 가능한지
- SCOPE_CYCLE_COUNTER 프로파일링 마커 추가
- 틱에서 분리 가능한 로직 → Timer 또는 이벤트 기반으로 전환 가능 여부

최적화 전후 예상 영향도 설명해줘.
```

### 예시 8: 테스트 코드 생성

```
@Source/BaseballManager/Core/Services/BMGachaService.h 에 대한
Automation Test를 작성해줘.

테스트 케이스:
- 등급별 확률이 합산 100%인지 검증
- 천장(pity) 시스템 정상 동작 (80연차 후 SSR 보장)
- 시드값 고정 시 결과 재현성 검증
- 재화 부족 시 에러 처리 검증
- TransactWriteItems 실패 시 롤백 확인

IMPLEMENT_SIMPLE_AUTOMATION_TEST 매크로 사용,
Source/BaseballManager/Tests/ 에 생성해줘.
```

---

## 7. Unreal MCP — AI가 에디터를 직접 조작하게 하기

### Unreal MCP란

MCP(Model Context Protocol)를 통해 Claude Code가 **Unreal Editor에 직접 명령을 보낼 수 있습니다**.  
코드 파일을 생성하는 것이 아니라, 에디터 안에서 액터 생성, 머티리얼 수정, 블루프린트 편집 등을 자동으로 수행합니다.

### 할 수 있는 것

- 액터 생성/삭제/트랜스폼 조작
- 블루프린트 생성 및 컴포넌트 추가
- 머티리얼/머티리얼 인스턴스 파라미터 수정
- 레벨 관리 (로드, 저장, 생성)
- 에셋 검색 및 의존성 조회
- Enhanced Input 설정
- 콘솔 명령 실행, 아웃풋 로그 읽기
- 애니메이션 블루프린트 스테이트 머신 편집

### 주요 MCP 프로젝트 (오픈소스)

| 프로젝트 | UE 버전 | 특징 |
|---|---|---|
| UnrealClaude (Natfii) | 5.7 | 에디터 내장 채팅, 20+ MCP 도구, UE 5.7 컨텍스트 |
| unreal-mcp (chongdashu) | 5.5 | Python MCP 서버, 범용 에디터 제어 |
| unreal-mcp (GenOrca) | 5.x | BP 노드 검사, 머티리얼 시스템, 레이캐스트 |
| ue-codegraph-mcp | 5.x | C++ 코드 인덱싱, 콜 그래프 분석, 심볼 검색 |

### 설치 예시 (unreal-mcp)

**1단계: Unreal 플러그인 설치**

프로젝트의 `Plugins/` 폴더에 플러그인을 복사하고, 에디터에서 활성화합니다.

**2단계: Claude Code에 MCP 서버 등록**

VS Code 터미널(`` Ctrl + ` `` 또는 상단 메뉴 **터미널 → 새 터미널**)에서:

```bash
claude mcp add unrealMCP -s project \
  -- uv --directory "C:\Projects\BaseballManager\Plugins\UnrealMCP\Python" \
  run unreal_mcp_server.py
```

**3단계: 연결 확인**

Claude Code 채팅 패널에서:
```
Unreal MCP로 현재 레벨의 액터 목록을 보여줘
```

### 코드 분석 MCP (ue-codegraph-mcp)

C++ 코드베이스를 인덱싱하여 Claude가 콜 그래프, 클래스 계층, #include 관계를 검색할 수 있게 합니다.

```bash
# 설치 및 인덱싱
npm install -g ue-codegraph-mcp
ue-graph index D:\Projects\BaseballManager\Source BaseballManager

# Claude Code에 등록
claude mcp add -s user ue-codegraph node "C:\path\to\ue-codegraph-mcp\build\index.js"
```

등록 후 Claude에게 이런 질문이 가능합니다:
```
SetHP를 호출하는 모든 함수를 찾아줘
ABMMatchSimulator 클래스의 상속 계층을 보여줘
BMPlayerState.h를 include하는 모든 파일을 알려줘
BlueprintCallable로 노출된 함수 목록을 보여줘
```

> ⚠️ Unreal MCP 프로젝트들은 대부분 아직 실험 단계(beta/alpha)입니다.  
> 프로덕션 레벨이나 중요 맵에서 사용하기 전에 **반드시 소스 컨트롤에 커밋**해두세요.

---

## 8. 효과적인 프롬프트 작성법

### 원칙: 코드 리뷰를 요청하듯 지시하라

여러분이 주니어 개발자에게 작업을 지시할 때를 떠올리세요. "투수 만들어"가 아니라, 클래스 설계와 제약 조건을 명확히 전달하겠죠.

### 나쁜 프롬프트 vs 좋은 프롬프트

**❌ 나쁨:**
```
투수 클래스 만들어줘
```

**✅ 좋음:**
```
투수 캐릭터 클래스를 만들어줘.

클래스:
- ABMPitcherCharacter : ABMBaseCharacter (기존 베이스 클래스 상속)
- UBMPitchingComponent : UActorComponent (투구 로직 담당)

GAS:
- UBMAttributeSet_Pitcher에 구위/제구/스태미나/멘탈 어트리뷰트
- 투구 어빌리티는 GameplayTag BM.Ability.Pitch.* 로 분류

리플리케이션:
- 투구 결과는 서버에서 판정, Client RPC로 연출 정보 전달
- 스태미나는 UPROPERTY(ReplicatedUsing=OnRep_Stamina)

기존 @Source/BaseballManager/Player/BMBaseCharacter.h 와
@Source/BaseballManager/Match/Abilities/ 패턴을 따라줘.
.h와 .cpp 쌍으로, Build.cs 의존성 추가도 포함해줘.
```

### 프롬프트 체크리스트 (UE 특화)

1. **클래스 계층** — 부모 클래스와 접두사 (A/U/F/E/I)
2. **매크로** — UPROPERTY, UFUNCTION 스펙파이어 (Replicated, BlueprintCallable 등)
3. **모듈** — 어떤 모듈에 배치할지, Build.cs 의존성
4. **리플리케이션** — 서버/클라이언트 실행 위치, 동기화 방식
5. **참조** — 기존 코드 패턴 (`@` 멘션)
6. **산출물** — .h + .cpp + Build.cs + 테스트 등 원하는 범위

### 점진적 작업 패턴

```
1단계: "경기 시뮬레이션 시스템의 클래스 다이어그램을 설계해줘. 코드 없이."
2단계: "핵심 데이터 구조(USTRUCT)와 인터페이스를 먼저 만들어줘."
3단계: "투구 판정 로직을 구현해줘."
4단계: "타격 판정 로직을 구현해줘. 투구 판정과 연동."
5단계: "리플리케이션을 추가해줘."
6단계: "Automation 테스트를 작성해줘."
```

---

## 9. 주의사항과 한계

### 소스 컨트롤은 생명줄입니다

Claude Code는 `.h`와 `.cpp`를 직접 수정합니다.

- **작업 전 반드시 커밋**하세요
- 대규모 리팩토링은 별도 브랜치에서 진행하세요
- `.uasset` 파일은 Claude가 수정하지 않지만, 코드 변경이 블루프린트 호환성을 깨뜨릴 수 있습니다

### AI가 모르는 것들

Claude Code는 강력하지만 **Unreal Editor를 실행하지 않습니다** (MCP 없이):

- 블루프린트 그래프의 시각적 구조를 모릅니다
- 에디터에서의 실제 컴파일 결과를 예측하지 못합니다
- 에셋 레퍼런스(.uasset)의 실제 내용을 읽지 못합니다
- 물리/렌더링의 실제 체감을 판단하지 못합니다
- Shipping 빌드에서의 성능 특성을 모릅니다
- 특정 플랫폼(모바일)의 제약사항을 완벽히 알지 못합니다

### AI 생성 UE C++ 코드의 검토 포인트

1. **UObject 라이프사이클**: GC에 의한 댕글링 포인터, TObjectPtr 사용 여부
2. **UPROPERTY 스펙파이어**: Replicated 빠짐, EditAnywhere vs EditDefaultsOnly 혼동
3. **GetLifetimeReplicatedProps**: 리플리케이션 프로퍼티 등록 누락
4. **메모리**: 게임스레드 외 접근, 매 틱 TArray 재할당
5. **IWYU**: 불필요한 #include, 전방 선언 가능한데 include한 경우
6. **모듈 의존성**: Build.cs에 의존성 추가 누락, 순환 참조
7. **네이밍**: 접두사 규칙 위반 (A/U/F/E/I/T)
8. **Hot Reload 호환**: UCLASS 구조 변경 시 에디터 재시작 필요 여부

### Unreal 특유의 함정들

- AI가 **STL 컨테이너**(std::vector, std::string)를 쓸 수 있습니다 → TArray, FString으로 교체
- AI가 **일반 C++ 예외처리**(try/catch)를 쓸 수 있습니다 → UE에서는 check/ensure/verify 사용
- AI가 **스마트 포인터를 혼동**할 수 있습니다 → TSharedPtr(non-UObject), TWeakObjectPtr(UObject) 구분
- AI가 **에디터 전용 코드**에 `#if WITH_EDITOR` 가드를 빠뜨릴 수 있습니다
- AI가 **Shipping 빌드**에서 제거되는 코드(UE_LOG 레벨, DrawDebug 등)를 고려하지 않을 수 있습니다

---

## 10. 레퍼런스

### 공식 문서

- Claude Code 공식 문서: [https://code.claude.com/docs](https://code.claude.com/docs)
- VS Code 확장: [https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)
- CLAUDE.md 작성 가이드: [https://code.claude.com/docs/en/claude-md](https://code.claude.com/docs/en/claude-md)

### Unreal 관련

- UnrealClaude (UE 5.7 에디터 통합): [https://github.com/Natfii/UnrealClaude](https://github.com/Natfii/UnrealClaude)
- Unreal MCP (에디터 제어): [https://github.com/chongdashu/unreal-mcp](https://github.com/chongdashu/unreal-mcp)
- Unreal MCP (GenOrca, BP 지원): [https://github.com/GenOrca/unreal-mcp](https://github.com/GenOrca/unreal-mcp)
- UE Code Graph MCP (C++ 코드 분석): [https://glama.ai/mcp/servers/@YomenStyle/ue-codegraph-mcp](https://glama.ai/mcp/servers/@YomenStyle/ue-codegraph-mcp)
- Claude Code UE C++ Skill: [https://fastmcp.me/skills/details/1072/unreal-engine-cpp-pro](https://fastmcp.me/skills/details/1072/unreal-engine-cpp-pro)

### 사내 자료

- 기획팀 Claude Code 매뉴얼: `vscode.md` (비개발자용)
- Unity 개발자 가이드: `unity-claude-code-guide.md`
- AWS 서버 개발자 가이드: `aws-server-claude-code-guide.md`
- 프로젝트 CLAUDE.md: 프로젝트 루트에 위치

---

> 📝 이 가이드에 대한 질문이나 개선 사항은 CEO 미니에게 전달해주세요.  
> AI 도구 도입 과정에서 발견한 팁이나 문제가 있으면 팀 내 공유 부탁드립니다.
