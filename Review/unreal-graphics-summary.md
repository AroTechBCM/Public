# 언리얼 그래픽 아티스트를 위한 Claude Code 가이드 요약

> 원본: `unreal-graphics-claude-code-guide.md`
> 대상: Unreal Engine 그래픽/테크니컬 아티스트 (경력 10년+)
> 버전: 1.0 (2026.03)

---

## 핵심 개념

경험이 쌓일수록 **에디터 밖의 코드**가 필요한 순간이 늘어난다. Claude Code는 그 순간을 해결한다 — HLSL, C++, Python을 한국어 설명만으로 생성.

---

## 설치

1. VS Code → `Ctrl+Shift+X` → **Claude Code (Anthropic)** 설치
2. **`.uproject` 파일이 있는 프로젝트 루트** 열기
3. `.claudeignore` 설정 (Binaries/, Content/, *.uasset, *.dll 등 제외)

---

## CLAUDE.md 핵심 내용 (그래픽 파이프라인)

| 항목 | 내용 |
|---|---|
| 렌더러 | Deferred (Lumen 미사용, 모바일 타겟) |
| 타겟 | Android(Vulkan), iOS(Metal), Windows(DX12) |
| 마스터 머티리얼 | MI_Character_Master, MI_Stadium_Master, MI_VFX_Master |
| 텍스처 네이밍 | T_{대상}_{타입} — _D, _N, _ORM, _E, _M, _A |
| 채널 패킹 | ORM (R=AO, G=Roughness, B=Metallic) |
| 셰이더 제한 | 모바일 픽셀 셰이더 128 인스트럭션 이하 |
| Python 스크립트 | Scripts/Editor/ 폴더 |

---

## 작업 도구 역할 분담

| 도구 | 역할 |
|---|---|
| Unreal Editor | 머티리얼 에디터, Niagara, 라이팅, 레벨 편집, 결과 확인 |
| VS Code (Claude Code) | HLSL 코드, Python 스크립트, C++ 코드 생성/수정 |

---

## 실전 프롬프트 예시 (8가지)

| 작업 | 요약 |
|---|---|
| 커스텀 HLSL | WPO 잔디 바람 효과, MPC 연동, 모바일 최적화 |
| 머티리얼 파라미터 제어 | UActorComponent + UCurveFloat + MPC 보간 |
| Python 에셋 일괄 처리 | 텍스처 임포트 설정 일괄 변경, CSV 백업 |
| Niagara HLSL 모듈 | 충격파 파티클, 속도/크기/색상 커브 |
| 포스트프로세스 | 방사형 블러 + 색수차, 샘플 수 파라미터화 |
| 머티리얼 인스턴스 일괄 생성 | CSV → 팀별 유니폼 MI 자동 생성 |
| 셰이더 최적화 | 텍스처 샘플 수, 분기 제거, half precision |
| 라이팅 프리셋 자동화 | JSON 프리셋 → 에디터 조명 일괄 적용 |

---

## Unreal MCP (그래픽 특화)

| 기능 | 활용 예시 |
|---|---|
| 머티리얼 인스턴스 파라미터 | "Roughness를 0.6으로 변경해줘" |
| 액터 배치 | "조명탑 4개를 대칭 배치해줘" |
| 에셋 검색 | "T_Stadium으로 시작하는 텍스처를 찾아줘" |
| 콘솔 명령 | "stat gpu 실행해줘" |

추천 MCP: `UnrealClaude`, `unreal-mcp (GenOrca)`, `CLAUDIUS (130+ 명령)`

---

## AI 생성 코드 검토 포인트

**HLSL:**
1. 인스트럭션 수 (모바일 128 제한 — 머티리얼 에디터 Stats에서 확인)
2. 텍스처 샘플러 수 (모바일 13개 제한)
3. float vs half 정밀도 (모바일에서 half가 효율적)
4. 분기 → `lerp + step`으로 대체 권장

**Python:**
1. `unreal.EditorAssetLibrary`, `unreal.MaterialEditingLibrary` 올바른 API 사용
2. `/Game/...` UE 에셋 경로 vs 파일 시스템 경로 혼동 주의
3. `unreal.EditorTransaction`으로 일괄 Undo 지원

---

## 주의사항

- MCP로 에디터 조작 전 **레벨과 에셋 저장** 필수
- Python 일괄 처리 스크립트에는 **백업 기능 포함** 요청
- Claude는 셰이더 컴파일 결과, 실제 렌더링 외형, 디바이스 성능을 알 수 없음
- 수학적으로 맞아도 시각적 결과는 에디터에서 반드시 확인
