# 🎨 언리얼 그래픽 아티스트를 위한 Claude Code 가이드
## AI 코딩 도구를 처음 도입하는 경력 테크니컬 아티스트용

> **대상**: Unreal Engine 그래픽/테크니컬 아티스트 (경력 10년 이상, AI 코딩 도구 미경험)  
> **프로젝트**: 야구매니저  
> **버전**: 1.0 (2026.03)  
> **작성**: KDM

---

## 목차

1. [왜 AI 코딩 도구인가](#1-왜-ai-코딩-도구인가)
2. [Claude Code 개요](#2-claude-code-개요)
3. [설치 및 세팅](#3-설치-및-세팅)
4. [CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기](#4-claudemd--ai에게-프로젝트-컨텍스트-주입하기)
5. [그래픽 작업 워크플로우](#5-그래픽-작업-워크플로우)
6. [실전 활용 예시](#6-실전-활용-예시)
7. [Unreal MCP — AI가 에디터를 직접 조작하게 하기](#7-unreal-mcp--ai가-에디터를-직접-조작하게-하기)
8. [효과적인 프롬프트 작성법](#8-효과적인-프롬프트-작성법)
9. [주의사항과 한계](#9-주의사항과-한계)
10. [레퍼런스](#10-레퍼런스)

---

## 1. 왜 AI 코딩 도구인가

### 10년 경력 그래픽 아티스트에게 AI 도구가 의미하는 것

여러분은 머티리얼 에디터에서 노드 수백 개를 엮어 복잡한 셰이더를 만들고, Niagara로 파티클 시스템을 설계하고, 라이팅 파이프라인을 최적화해본 분들입니다. 여러분이 하는 대부분의 작업은 에디터의 비주얼 노드 그래프 안에서 이루어지죠.

그런데 경험이 쌓일수록 **에디터 밖의 코드**가 필요한 순간이 늘어납니다:

| 상황 | 필요한 것 |
|---|---|
| 커스텀 셰이더 노드가 필요할 때 | HLSL Custom Expression 또는 C++ Material Expression |
| 머티리얼 파라미터를 게임 로직과 연동할 때 | C++ 또는 Blueprint에서 MID/MPC 제어 |
| 에셋을 수백 개 일괄 수정할 때 | Python Editor Script 또는 C++ 에디터 유틸리티 |
| Niagara 모듈을 직접 만들 때 | HLSL 코드 또는 Scratch Pad 스크립트 |
| 포스트프로세스 커스텀 패스가 필요할 때 | C++ 렌더링 코드 또는 커스텀 노드 |
| 셰이더 성능 프로파일링 자동화 | Python 스크립트 + stat 명령어 |
| 팀 전용 머티리얼 라이브러리 도구 | 에디터 플러그인 (Slate/UMG + C++) |

**Claude Code는 이 "코드가 필요한 순간"을 해결해줍니다.** 여러분은 원하는 것을 한국어로 설명하고, Claude가 HLSL, C++, Python 스크립트를 생성합니다. 코드를 처음부터 작성할 줄 몰라도 됩니다. 결과물을 읽고 판단할 수 있으면 충분합니다.

### 어떤 것이 아닌지

- ❌ 머티리얼 에디터의 노드 그래프를 대신 만들어주는 것 → 아닙니다 (MCP 제외)
- ❌ 텍스처나 3D 모델을 생성해주는 것 → 아닙니다
- ❌ 라이팅을 자동으로 세팅해주는 것 → 아닙니다

### 어떤 것인지

- ✅ 커스텀 셰이더 코드(HLSL)를 설명만으로 생성
- ✅ 에디터 자동화 스크립트(Python/C++) 생성
- ✅ 머티리얼 파라미터 제어 코드를 기존 패턴에 맞게 생성
- ✅ 기존 셰이더 코드의 문제점 분석과 최적화 제안
- ✅ MCP를 통해 머티리얼/머티리얼 인스턴스를 에디터에서 직접 조작
- ✅ Niagara 모듈 스크립트, 커스텀 렌더 패스 코드 생성

---

## 2. Claude Code 개요

### Claude Code란

Anthropic이 만든 **에이전틱 코딩 도구**입니다. VS Code 확장으로 설치하며, 프로젝트 폴더 전체를 컨텍스트로 활용합니다.

핵심 특징:

- **파일 시스템 접근**: `.usf`, `.ush`, `.h`, `.cpp`, `.py`, `.ini` 등 모든 파일을 직접 읽고 쓸 수 있음
- **CLAUDE.md**: 프로젝트 루트의 규칙 파일을 자동으로 읽어 매번 컨텍스트 설명 불필요
- **diff 기반 수정**: 모든 파일 수정을 diff로 보여주고 Accept/Reject 선택 가능
- **MCP 연동**: Unreal MCP로 에디터의 머티리얼, 액터, 에셋을 직접 제어 가능

### 비용

Claude Max 구독 또는 Anthropic API 키가 필요합니다. 회사 계정 정보는 팀장에게 문의하세요.  
세션 중 `/cost`를 입력하면 비용을 확인할 수 있습니다.

---

## 3. 설치 및 세팅

### 3-1. VS Code + 확장 설치

여러분이 평소 사용하는 IDE가 무엇이든, Claude Code 작업은 **VS Code에서** 진행합니다.

1. VS Code가 없다면: [https://code.visualstudio.com](https://code.visualstudio.com) 에서 다운로드
2. VS Code에서 `Ctrl + Shift + X` → **"Claude Code"** 검색 → Publisher: **Anthropic** Install
3. 왼쪽 사이드바에 ✱ (Spark) 아이콘이 나타나면 설치 완료
4. 아이콘을 처음 클릭하면 Anthropic 로그인 화면이 뜹니다. 회사 계정으로 로그인

### 3-2. 프로젝트 폴더 열기

VS Code에서 **`.uproject` 파일이 있는 프로젝트 루트**를 엽니다.

```
상단 메뉴: 파일(File) → 폴더 열기(Open Folder) → 프로젝트 루트 선택
```

> ⚠️ `Content/` 폴더나 `Source/` 폴더만 열면 안 됩니다.  
> 프로젝트 전체 구조를 인식해야 합니다.

### 3-3. .claudeignore

Unreal 프로젝트는 바이너리 파일이 매우 많습니다. 아래 파일을 프로젝트 루트에 `.claudeignore`라는 이름으로 만드세요:

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

# 에셋 (바이너리)
Content/
*.uasset
*.umap

# 바이너리
*.dll
*.lib
*.pdb
*.exe
*.o
*.so
```

> 💡 `.uasset`은 바이너리이므로 Claude가 읽을 수 없습니다.  
> 에셋 구조, 머티리얼 계층, 텍스처 컨벤션 등은 CLAUDE.md에 텍스트로 기술하세요.

---

## 4. CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기

### CLAUDE.md가 하는 일

프로젝트 루트에 `CLAUDE.md`를 놓으면, Claude Code가 **모든 대화 시작 시 이 파일을 자동으로 읽습니다**. 매번 "우리 프로젝트는 URP... 아니 UE5의 Lumen을 쓰고, 머티리얼은..." 같은 설명을 반복할 필요가 없습니다.

### 야구매니저 그래픽 파이프라인용 CLAUDE.md 예시

```markdown
# 야구매니저 — 그래픽 파이프라인 규칙

## 프로젝트 정보
- 엔진: Unreal Engine 5.4
- 렌더러: Deferred Rendering (Lumen 미사용, 모바일 타겟 고려)
- 타겟 플랫폼: Android (Vulkan), iOS (Metal), Windows (DX12)
- 품질 티어: Low / Medium / High (Scalability Group으로 관리)

## 렌더링 파이프라인
- Global Illumination: Screen Space GI (모바일), Distance Field AO (PC)
- 반사: Screen Space Reflections + Reflection Capture
- 그림자: Cascaded Shadow Maps (3단계)
- 안티앨리어싱: TAA (PC), FXAA (모바일)
- 포스트프로세스: Bloom, Color Grading, Vignette, DOF(경기 중만)

## 머티리얼 컨벤션
- 마스터 머티리얼 사용:
  - MI_Character_Master: 선수/캐릭터용
  - MI_Stadium_Master: 경기장 환경용
  - MI_UI_Master: UI 요소용
  - MI_VFX_Master: 이펙트용
- 머티리얼 인스턴스 네이밍: MI_{대상}_{변형}_{세부}
  - 예: MI_Character_Home_Uniform, MI_Stadium_Grass_Day
- 텍스처 네이밍: T_{대상}_{타입}
  - 타입: _D (Diffuse/BaseColor), _N (Normal), _ORM (Occlusion/Roughness/Metallic),
    _E (Emissive), _M (Mask), _A (Alpha)
  - 예: T_Character_D, T_Stadium_Grass_N
- 텍스처 해상도: 캐릭터 2048, 환경 1024, UI 512 (모바일 기준)
- 채널 패킹: ORM 텍스처 (R=AO, G=Roughness, B=Metallic)
- Material Domain: Surface (기본), Post Process, UI, Decal, Volume

## 셰이더 규칙
- Custom HLSL은 Shaders/ 폴더에 .usf/.ush로 관리
- 커스텀 노드 사용 시 주석으로 입력/출력 설명 필수
- 모바일 셰이더: SM5 기능(tessellation 등) 사용 금지
- 인스트럭션 카운트 제한: 모바일 기준 픽셀 셰이더 128 이하 권장
- 텍스처 샘플러: 머티리얼당 16개 이하 (모바일 13개 이하)

## Niagara 규칙
- 이펙트 네이밍: NS_{대상}_{동작} (예: NS_Ball_Trail, NS_Bat_Impact)
- 이미터 타입: GPU 우선, Sprite/Mesh Renderer
- 모바일: 파티클 수 PC의 50%로 Scalability 설정
- 커스텀 모듈: Scratch Pad → 검증 후 HLSL 모듈로 전환

## 라이팅 규칙
- 경기장 조명: Directional Light (태양) + Sky Light + 조명탑(Point/Spot)
- 낮/밤 전환: Material Parameter Collection (MPC_TimeOfDay)
- 베이크: 경기장 고정 오브젝트만 Static 라이팅
- 동적 오브젝트(선수, 공): 전부 Movable

## 에셋 자동화
- Python Editor Script 위치: Scripts/Editor/
- 일괄 처리 도구: Scripts/Editor/BatchTools/
- 텍스처 임포트 규칙: Scripts/Editor/TextureImportRules.py

## 폴더 구조 (Content)
Content/
├── Characters/
│   ├── Meshes/
│   ├── Materials/
│   ├── Textures/
│   └── Animations/
├── Environment/
│   ├── Stadium/
│   ├── Props/
│   └── Landscape/
├── VFX/
│   ├── Niagara/
│   ├── Materials/
│   └── Textures/
├── UI/
├── PostProcess/
└── MasterMaterials/
```

## 응답 규칙
- 모든 응답, 주석은 한국어
- HLSL 코드 내 주석도 한국어
- 셰이더 코드 생성 시 입력/출력 핀 설명을 주석으로 포함할 것
- 새 파일 생성 시 상단에 용도 요약 주석
```

---

## 5. 그래픽 작업 워크플로우

### 기본 작업 흐름

```
VS Code에서 Claude Code 패널 열기 (왼쪽 사이드바 ✱ 아이콘 클릭)
        ↓
원하는 것을 한국어로 설명 (+ 관련 파일 @멘션)
        ↓
Claude가 HLSL, C++, Python 등 코드를 생성
        ↓
diff 화면에서 확인 → Accept 또는 Reject 클릭
        ↓
Unreal Editor에서 결과 확인 (머티리얼 리컴파일, 스크립트 실행 등)
```

### 작업 도구 병행 패턴

| 도구 | 역할 |
|---|---|
| **Unreal Editor** | 머티리얼 에디터, Niagara, 라이팅, 레벨 편집, 결과 확인 |
| **VS Code (Claude Code)** | HLSL 코드, Python 스크립트, C++ 코드 생성/수정 |

대부분의 시각적 작업은 여전히 Unreal Editor에서 합니다.  
Claude Code는 **코드가 필요한 순간**에 옆에서 보조하는 역할입니다.

### @ 멘션으로 파일 참조하기

채팅 입력창에 `@`를 입력하면 프로젝트 내 파일 목록이 나타납니다.  
클릭으로 선택하면 Claude가 해당 파일의 내용을 직접 읽습니다.

```
@Shaders/Stadium/StadiumGrass.usf 이 셰이더에 
바람에 의한 잔디 움직임 효과를 추가해줘.
World Position Offset을 사용하고, 
풍향/풍속은 MPC에서 가져오도록 해줘.
```

---

## 6. 실전 활용 예시

### 예시 1: 커스텀 HLSL 셰이더 작성

```
경기장 잔디용 커스텀 셰이더를 HLSL로 만들어줘.

요구사항:
- World Position Offset으로 바람에 의한 흔들림
- 풍향/풍속은 MPC_TimeOfDay의 WindDirection/WindSpeed 파라미터에서 가져오기
- 버텍스 컬러 R 채널을 마스크로 사용 (뿌리 부분은 고정)
- sin파 기반 움직임 + 약간의 노이즈 (자연스러움)
- 거리에 따라 움직임 감쇠 (LOD 최적화)

Shaders/Stadium/GrassWind.usf 파일로 만들어줘.
머티리얼 에디터 Custom 노드에 넣을 수 있게 
입력/출력 핀을 주석으로 명확히 표기해줘.
모바일에서 돌아가야 하니까 인스트럭션 수를 최소화해줘.
```

### 예시 2: 머티리얼 파라미터 제어 C++ 코드

```
낮/밤 전환 시 머티리얼을 동적으로 제어하는 시스템을 만들어줘.

요구사항:
- UBMTimeOfDayController : UActorComponent
- MPC_TimeOfDay의 파라미터를 시간에 따라 보간:
  - SunIntensity (0.0 ~ 1.0)
  - SkyColor (FLinearColor)
  - StadiumLightIntensity (야간 경기 시 1.0, 주간 0.0)
  - WindDirection (FVector2D)
  - WindSpeed (0.0 ~ 1.0)
- UCurveFloat로 시간별 보간 커브 참조
- Blueprint에서 시간을 설정할 수 있게 BlueprintCallable

기존 @Source/BaseballManager/Core/ 의 컴포넌트 패턴을 따라줘.
.h와 .cpp 쌍으로 만들어줘.
```

### 예시 3: Python 에디터 스크립트 (에셋 일괄 처리)

```
프로젝트 내 모든 캐릭터 텍스처의 임포트 설정을 일괄 변경하는 
Python Editor Script를 만들어줘.

조건:
- Content/Characters/Textures/ 하위의 모든 텍스처
- _D (Diffuse) 텍스처: Compression = BC7, sRGB = True
- _N (Normal) 텍스처: Compression = BC5, sRGB = False, Flip Green Channel
- _ORM 텍스처: Compression = BC7, sRGB = False
- 해상도가 4096 이상이면 Max Texture Size를 2048로 제한
- LOD Bias: 모바일에서 1단계 낮게

변경 전에 기존 설정을 CSV로 백업해줘.
Scripts/Editor/BatchTools/FixCharacterTextures.py 에 저장해줘.
```

### 예시 4: Niagara 커스텀 모듈 (HLSL)

```
공이 배트에 맞을 때 충격파 이펙트용 Niagara HLSL 모듈을 만들어줘.

요구사항:
- 파티클 초기 속도: 충돌 법선 방향으로 발사 (cone 형태)
- 속도 감쇠: 거리에 따라 exp 감쇠
- 크기: 수명에 따라 커지다가 줄어듦 (ease-out-in 커브)
- 색상: 흰색 → 노란색 → 투명 (수명 기반)
- 스프라이트 회전: 랜덤 + 속도 방향 정렬
- GPU Sim 기준

Niagara Scratch Pad에 넣을 수 있는 형태로 작성하고,
입력 파라미터와 출력 파라미터를 주석으로 정리해줘.
```

### 예시 5: 포스트프로세스 머티리얼 코드

```
경기 하이라이트 리플레이에서 사용할 속도감 표현 
포스트프로세스 머티리얼을 만들어줘.

요구사항:
- 방사형 블러 (화면 중앙에서 바깥으로)
- 블러 강도: Scalar 파라미터로 조절 (0이면 효과 없음)
- 화면 가장자리에 비네팅 + 색수차(Chromatic Aberration)
- 색수차 강도도 파라미터로 분리
- Post Process Material Domain, Blendable Location: Before Translucency

Custom 노드에 넣을 HLSL 코드로 작성해줘.
SceneTexture:PostProcessInput0에서 샘플링하는 패턴이어야 해.
모바일에서도 돌아가야 하니까 루프 횟수를 파라미터화해줘
(PC: 16샘플, 모바일: 8샘플).
```

### 예시 6: 에디터 유틸리티 (머티리얼 인스턴스 일괄 생성)

```
선수 유니폼 머티리얼 인스턴스를 팀별로 일괄 생성하는 
Python Editor Script를 만들어줘.

입력: CSV 파일 (팀명, 메인컬러 HEX, 서브컬러 HEX, 로고텍스처 경로)

동작:
1. MI_Character_Master를 부모로 머티리얼 인스턴스 생성
2. 팀별 컬러 파라미터 설정 (BaseColor, AccentColor)
3. 로고 텍스처 파라미터 연결
4. Content/Characters/Materials/Uniforms/{팀명}/ 폴더에 저장
5. 네이밍: MI_Uniform_{팀명}_Home, MI_Uniform_{팀명}_Away

CSV 예시:
```
TeamName,MainColor,SubColor,LogoTexture
Tigers,#FF6B00,#000000,/Game/Characters/Textures/Logos/T_Logo_Tigers
Eagles,#1B4D3E,#FFD700,/Game/Characters/Textures/Logos/T_Logo_Eagles
```

Scripts/Editor/BatchTools/GenerateUniformMaterials.py 에 저장해줘.
```

### 예시 7: 셰이더 성능 분석 / 최적화

```
@Shaders/Characters/CharacterSkin.usf 를 분석해줘.

확인할 것:
- 텍스처 샘플 수가 모바일 제한(13개)을 넘지 않는지
- 분기(if/branch)가 있다면 static branch로 변환 가능한지
- pow() 등 비용이 높은 연산을 근사식으로 대체할 수 있는지
- 중복 연산이 있는지
- half precision으로 변환 가능한 변수가 있는지
- 모바일 / PC 분기 (#if MOBILE_PROFILE) 가 필요한 곳

최적화 전후 예상 인스트럭션 차이도 설명해줘.
```

### 예시 8: 라이팅 데이터 자동화

```
경기장 조명 프리셋을 시간대별로 관리하는 Python 스크립트를 만들어줘.

요구사항:
- JSON 파일로 프리셋 정의:
  - "DayGame": 태양 고도 60°, 강도 8.0, 조명탑 OFF
  - "NightGame": 태양 고도 -10°, 강도 0.1, 조명탑 ON, 조명탑 강도 50000lux
  - "Sunset": 태양 고도 15°, 강도 3.0, 색온도 3200K
- 에디터에서 실행하면 현재 레벨의 Directional Light, Sky Light,
  조명탑(SpotLight 배열)에 프리셋을 적용
- MPC_TimeOfDay 파라미터도 프리셋에 맞게 갱신
- 보간 모드 옵션: 즉시 적용 / N초에 걸쳐 보간

Scripts/Editor/LightingPresets.py 에 저장하고,
프리셋 JSON은 Config/LightingPresets.json 에 저장해줘.
```

---

## 7. Unreal MCP — AI가 에디터를 직접 조작하게 하기

### Unreal MCP란

MCP(Model Context Protocol)를 통해 Claude Code가 **Unreal Editor를 직접 조작**할 수 있습니다.  
코드 파일을 만드는 것을 넘어서, 에디터 안에서 머티리얼 파라미터 변경, 액터 배치, 에셋 검색 등을 수행합니다.

### 그래픽 아티스트에게 유용한 MCP 기능

| 기능 | 활용 예시 |
|---|---|
| 머티리얼 인스턴스 파라미터 수정 | "MI_Stadium_Grass_Day의 Roughness를 0.6으로 변경해줘" |
| 머티리얼 인스턴스 생성 | "MI_Character_Master로부터 새 인스턴스를 만들어줘" |
| 액터 트랜스폼 조작 | "조명탑 4개를 경기장 모서리에 대칭 배치해줘" |
| 에셋 검색 | "T_Stadium으로 시작하는 모든 텍스처를 찾아줘" |
| 에셋 참조/의존성 조회 | "MI_Character_Home이 참조하는 텍스처 목록을 보여줘" |
| 콘솔 명령 실행 | "stat gpu 실행해줘" |
| Sequencer 제어 | "현재 시퀀스를 렌더링해줘" |

### 주요 MCP 프로젝트

| 프로젝트 | 특징 |
|---|---|
| UnrealClaude (Natfii) | UE 5.7, 에디터 내장 채팅, 머티리얼 도구 포함 |
| unreal-mcp (GenOrca) | 머티리얼 파라미터 제어, 에셋 관리, BP 노드 검사 |
| unreal-mcp (chongdashu) | 액터/블루프린트/머티리얼/입력 제어 |
| CLAUDIUS | JSON 명령 기반, 130+ 에디터 명령, Sequencer/PCG 지원 |

### 설치 방법 (unreal-mcp 기준)

1. 프로젝트 `Plugins/` 폴더에 MCP 플러그인 복사 → 에디터에서 활성화
2. VS Code 터미널(상단 메뉴 **터미널 → 새 터미널**)에서 MCP 서버 등록:

```bash
claude mcp add unrealMCP -s project \
  -- uv --directory "C:\Projects\BaseballManager\Plugins\UnrealMCP\Python" \
  run unreal_mcp_server.py
```

3. Claude Code 채팅에서 확인:
```
Unreal MCP로 현재 레벨의 모든 Light 액터를 나열해줘
```

### MCP 활용 예시 (그래픽 작업)

```
Unreal MCP를 사용해서:
Content/Characters/Materials/ 안의 모든 머티리얼 인스턴스에서
Roughness 파라미터 값을 조회해줘.
0.8 이상인 것이 있으면 0.7로 변경해줘.
```

```
현재 레벨의 모든 SpotLight를 찾아서
Intensity를 현재 값의 120%로 올려줘.
Attenuation Radius는 변경하지 마.
```

> ⚠️ MCP로 에디터를 직접 수정하기 전에 **반드시 레벨을 저장**하세요.  
> 실수로 라이팅 세팅이 바뀌면 되돌리기 어려울 수 있습니다.

---

## 8. 효과적인 프롬프트 작성법

### 원칙: 아트 디렉터가 피드백하듯 구체적으로

여러분이 주니어 아티스트에게 작업을 지시할 때, "예쁘게 해줘"라고 하지 않겠죠. 같은 원칙입니다.

### 나쁜 프롬프트 vs 좋은 프롬프트

**❌ 나쁨:**
```
잔디 셰이더 만들어줘
```

**✅ 좋음:**
```
경기장 잔디용 World Position Offset 셰이더를 HLSL로 만들어줘.

기술 스펙:
- 바람 방향/속도는 MPC_TimeOfDay에서 가져오기
- 버텍스 컬러 R 채널을 움직임 마스크로 사용 (0=고정, 1=최대 움직임)
- 움직임: sin(Time * Speed + WorldPos.x * Frequency) 기반
- 약간의 Perlin 노이즈 추가 (자연스러움)
- LOD: 카메라 거리 5000 이상이면 움직임 감쇠

제약:
- 모바일(Vulkan ES3.1)에서 동작해야 함
- 인스트럭션 20개 이내 권장
- half precision 사용

출력:
- Shaders/Stadium/GrassWind.usf 파일
- 입력/출력 핀 주석 포함
```

### 프롬프트 체크리스트 (그래픽 특화)

1. **시각적 목표** — 어떤 효과/결과를 원하는지
2. **기술 스펙** — 사용할 기법(WPO, Custom Node, PP 등), 입출력 데이터
3. **파라미터** — 조절 가능해야 하는 값, 기본값
4. **타겟 플랫폼** — 모바일 제약, SM 버전, 인스트럭션 제한
5. **참조** — 기존 셰이더나 머티리얼 패턴 (`@` 멘션)
6. **출력 위치** — 어떤 폴더에 어떤 파일명으로 생성할지

---

## 9. 주의사항과 한계

### 소스 컨트롤은 생명줄입니다

- 코드 파일 수정 전에 **반드시 커밋**하세요
- MCP로 에디터를 조작하기 전에 **레벨과 에셋을 저장**하세요
- Python 일괄 처리 스크립트는 **백업 기능을 포함**해달라고 요청하세요

### AI가 모르는 것들

Claude Code는 **Unreal Editor를 실행하지 않습니다** (MCP 없이):

- 머티리얼 에디터의 노드 그래프를 볼 수 없습니다
- 셰이더 컴파일 결과(실제 인스트럭션 카운트)를 알 수 없습니다
- 텍스처/메시의 실제 외형을 볼 수 없습니다
- 라이팅 빌드 결과를 예측하지 못합니다
- 특정 디바이스에서의 실제 렌더링 성능을 알 수 없습니다
- Niagara 노드 그래프 구조를 읽지 못합니다

**결론**: Claude가 생성한 셰이더/머티리얼 코드는 반드시 에디터에서 시각적으로 확인하세요.  
수학적으로 맞더라도 눈으로 볼 때 원하는 결과가 아닐 수 있습니다.

### AI 생성 HLSL 코드의 검토 포인트

1. **인스트럭션 수**: 모바일 기준 초과 여부 (머티리얼 에디터 Stats에서 확인)
2. **텍스처 샘플러 수**: 모바일 13개 제한
3. **정밀도**: float vs half, 모바일에서 half가 더 효율적
4. **분기**: if/else 대신 lerp + step 사용 권장 (GPU 분기 비용)
5. **UE 매크로**: Material 표현식에서 사용 가능한 함수인지 (예: GetActorPosition은 Custom 노드에서 못 씀)
6. **좌표계**: UE는 Z-up, 좌표 혼동 주의

### AI 생성 Python 스크립트의 검토 포인트

1. **Editor API**: `unreal.EditorAssetLibrary`, `unreal.MaterialEditingLibrary` 등 올바른 API 사용
2. **경로**: `/Game/...` 형식 UE 에셋 경로 vs 실제 파일 시스템 경로 혼동
3. **대량 처리**: `unreal.ScopedSlowTask`로 진행률 표시 포함 여부
4. **Undo 지원**: `unreal.EditorTransaction`으로 일괄 Undo 가능하게

---

## 10. 레퍼런스

### 공식 문서

- Claude Code 공식 문서: [https://code.claude.com/docs](https://code.claude.com/docs)
- VS Code 확장: [https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)
- CLAUDE.md 작성 가이드: [https://code.claude.com/docs/en/claude-md](https://code.claude.com/docs/en/claude-md)

### Unreal MCP

- UnrealClaude (UE 5.7 에디터 통합): [https://github.com/Natfii/UnrealClaude](https://github.com/Natfii/UnrealClaude)
- Unreal MCP (머티리얼/에셋 지원): [https://github.com/GenOrca/unreal-mcp](https://github.com/GenOrca/unreal-mcp)
- CLAUDIUS (130+ 에디터 명령, Sequencer): [https://claudiuscode.com](https://claudiuscode.com)

### 사내 자료

- 기획팀 Claude Code 매뉴얼: `vscode.md` (비개발자용)
- 언리얼 개발자 가이드: `unreal-claude-code-guide.md` (C++ 개발자용)
- 프로젝트 CLAUDE.md: 프로젝트 루트에 위치

---

> 📝 이 가이드에 대한 질문이나 개선 사항은 CEO 미니에게 전달해주세요.  
> AI 도구 도입 과정에서 발견한 팁이나 문제가 있으면 팀 내 공유 부탁드립니다.
