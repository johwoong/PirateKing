# 해적왕 키우기 — 개발 초기 세팅 가이드

- 작성일: 2026-11-12
- "확인 필요"라고 적은 항목은 웹 검색 결과만 보고 적은 것으로, 직접 설치하거나 공식 문서 원문으로 검증하지 않았다.

## 현재 상태

| 항목 | 상태 |
|---|---|
| Unity | 6000.3.24f1, URP 17.3 기본 템플릿. 스크립트 없음 |
| 설치된 패키지 | Input System, uGUI(TextMeshPro 포함), AI Navigation, Timeline, Test Framework |
| Git | `PirateKing` 폴더에 저장소 생성함(커밋 없음). `.gitignore`, `.gitattributes` 작성함 |
| Notion, Tripo, Unity MCP | 연결 안 됨 |
| Google Sheets, UGS | 만들지 않음 |

**주의:** `C:\` 최상위에도 Git 저장소(`C:\.git`)가 있다. 드라이브 전체가 저장소로 잡혀 있어 의도한 것이 아닐 가능성이 높다. 이 프로젝트는 이제 자체 저장소를 쓰므로 영향은 없지만, 필요 없는 것이면 `C:\.git` 폴더를 지우는 것을 권한다. 지우기 전에 안에 보존할 커밋이 없는지 확인한다.

## 1. 원본 규칙

같은 내용을 두 곳에 적으면 반드시 어긋난다. 종류별로 원본을 하나만 둔다.

| 종류 | 원본 | 다른 곳에서는 |
|---|---|---|
| 기획 설명(글, 그림, 결정 기록) | Notion | `Docs/`는 초안과 백업 |
| 수치 데이터(능력치, 스테이지, 확률) | Google Sheets | Notion에는 숫자를 복사하지 않고 시트 링크만 |
| 코드, 시트에서 내보낸 데이터 | Git | |
| 영입 확률, 상점 상품, 보상량 | UGS(서버) | 시트에서 내보내 서버에 등록. 기기 안의 값은 표시용 |
| 3D 원본과 콘셉트 그림 | Git LFS | |

## 2. Git

1. 첫 커밋 전에 `git status`로 `Library`, `Temp`, `Logs`, `UserSettings`가 목록에 없는지 확인한다(현재 정상).
2. `git lfs install`을 한 번 실행한다. `fbx, glb, obj, psd, tga, exr, wav, mp3`가 LFS로 들어간다.
3. 원격 저장소는 비공개로 만든다. GitHub 무료 계정의 LFS 용량은 작으므로 3D 원본이 늘면 용량을 확인한다(확인 필요).
4. 서명 키(`*.keystore`), API 키, 서비스 계정 JSON은 커밋하지 않는다. `.gitignore`에 넣어 두었다. 서명 키는 잃어버리면 앱을 업데이트할 수 없으니 별도로 백업한다.

## 3. Unity 프로젝트

**빌드 설정**

| 항목 | 값 |
|---|---|
| 플랫폼 | Android |
| 화면 방향 | Portrait 고정 |
| Scripting Backend | IL2CPP |
| Target Architecture | ARM64 |
| 타깃 API 수준 | Play 최신 요구치(확인 필요) |
| 패키지 이름 | `com.<개발자명>.pirateking` 형식. 출시 후 변경 불가 |
| 프레임 | 60 고정 시도, 발열이 심하면 30 선택 옵션 |

**폴더 구조**

```
Assets/_Project/
  Art/         Characters/  Enemies/  Environment/  VFX/  UI/
  Audio/
  Data/        Generated/   (시트에서 만든 ScriptableObject, 직접 수정 금지)
  Prefabs/
  Scenes/      Boot  Lobby  Battle
  Scripts/     Core/  Battle/  Crew/  Gacha/  Stage/  UI/  Data/  Net/  Editor/
  Settings/
```

- 외부 에셋은 `Assets/` 바로 아래에 두고 직접 만든 것은 `_Project/` 안에만 둔다.
- 스크립트 영역마다 asmdef를 두어 컴파일 시간을 줄이고 의존 방향을 고정한다(UI → 게임 로직 → Core).
- 씬은 3개: Boot(로그인, 데이터 로드), Lobby(모든 메뉴), Battle(전투).

**패키지**

| 추가 | 용도 |
|---|---|
| Cinemachine | 선장을 따라가는 탑다운 카메라 |
| Authentication, Cloud Code, Cloud Save, Remote Config | 서버(2단계부터) |
| In-App Purchasing | 인앱결제(4단계) |
| 광고 SDK | Unity LevelPlay 또는 Google AdMob 중 선택(4단계, 미확정) |
| Addressables | 선원 50종을 필요할 때만 불러오기(3단계) |

제거: Visual Scripting, Multiplayer Center(쓰지 않음). `com.unity.pipeline`은 용도를 확인하지 못했으므로 그대로 둔다.

서버·결제·광고 패키지는 해당 단계가 될 때 넣는다. 처음부터 넣으면 빌드 시간과 설정만 늘어난다.

## 4. Claude Code와 Unity 연결

| 작업 | MCP 없이 | MCP 연결 시 |
|---|---|---|
| C# 스크립트 작성·수정 | 가능 | 가능 |
| 컴파일 오류 확인, 테스트 실행 | 가능(Unity 배치 모드. 에디터를 닫아야 함) | 에디터를 켠 채로 가능 |
| 씬에 오브젝트 배치, 프리팹 생성, 인스펙터 값 수정 | 불가(에디터 스크립트를 짜서 우회) | 가능 |
| 콘솔 로그 읽기, 플레이 모드 실행 | 불가 | 가능 |

연결 방법 후보(확인 필요):

1. **Unity 공식:** AI Assistant 패키지(`com.unity.ai.assistant`)에 MCP 서버가 들어 있고 Claude Code를 지원한다고 안내되어 있다. 다만 공식 MCP가 중단 예정이고 Unity CLI로 옮겨 간다는 외부 글이 있다. Unity 문서에서 현재 권장 방식을 먼저 확인한다.
2. **커뮤니티:** `MCP for Unity`(CoplayDev, MIT 라이선스). Unity 6을 지원하며 도구 수가 많다.

어느 쪽이든 순서는 같다: Unity에 패키지 설치 → 에디터의 설정 창에서 Claude Code 연결 → 프로젝트 폴더에서 Claude Code 실행 → `/mcp`로 연결 확인.

## 5. Notion

1. 터미널에서 실행: `claude mcp add --transport http notion https://mcp.notion.com/mcp`
2. Claude Code 안에서 `/mcp` → notion 선택 → 브라우저에서 로그인하고 접근할 페이지를 허용한다.
3. 연결 후 Claude에게 `Docs/`의 문서를 Notion에 올리라고 요청한다.

**페이지 구조 제안**

```
해적왕 키우기
├─ 게임 기획서 (GDD)        ← 이후 원본
├─ 결정 기록 (데이터베이스)   날짜 / 결정 / 이유 / 영향 범위
├─ 작업 보드 (데이터베이스)   할 일 / 진행 중 / 완료, 개발 단계별 보기
├─ 선원 도감 (데이터베이스)   50종의 콘셉트·제작 상태(콘셉트/모델/리깅/Unity 적용)
├─ UI 화면 설계              와이어프레임과 화면별 메모
└─ 링크 모음                 데이터 시트, UGS 대시보드, Play Console
```

- 와이어프레임 PNG는 Notion에 올릴 때 파일 업로드가 MCP로 되지 않으면 직접 끌어다 넣어야 한다(확인 필요).
- 선원 도감에는 설명과 제작 상태만 둔다. 능력치는 시트에만 둔다.

## 6. Google Sheets 데이터 관리

**파일 1개(`PirateKing_Data`)에 탭 10개**

| 탭 | 내용 | 주요 열 |
|---|---|---|
| Captain | 선장 3명 | id, name_key, weapon, hp, atk, def, atk_speed, range, skill1, skill2, ult |
| Crew | 선원 50종 | id, name_key, position, role, grade, hp, atk, def, atk_speed, range, skill, passive, prefab |
| Skill | 스킬 정의 | id, type, coef, cooldown, range, target, effect, value, duration |
| Enemy | 적 16종 | id, type, hp, atk, def, move_speed, skill, prefab |
| Stage | 스테이지 20개 | id, sea, order, recommend_power, time_limit, gold, exp, first_clear_gem |
| StageWave | 구역별 적 배치 | stage_id, zone, enemy_id, count, hp_scale, atk_scale |
| Gacha | 등급 확률과 대상 목록 | grade, rate, crew_id, weight |
| LevelCurve | 레벨·승급 비용 | level, need_gold, need_exp, stat_scale |
| Shop | 상품 | id, type, price_krw, gem, limit |
| Text | 화면 문구 | key, ko, en |

**작성 규칙**

- 1행은 열 이름(영문 소문자), 2행은 한글 설명, 3행부터 데이터.
- 모든 행에 `id`가 있다. 한 번 쓴 id는 바꾸거나 재사용하지 않는다.
- 열 이름이 `#`으로 시작하면 메모 열로 보고 가져오지 않는다.
- 수식으로 계산한 열을 써도 되지만 가져올 때는 결과값만 읽는다.
- 화면에 나오는 글자는 시트에 직접 쓰지 않고 `Text` 탭의 key를 참조한다(나중에 영어 추가 대비).

**시트 → Unity 흐름**

```
시트 수정 → Unity 메뉴 [Tools > Data > Import] → CSV 내려받기 → 검증 → ScriptableObject 생성(Data/Generated) → Git 커밋
```

- 게임 실행 중에는 시트를 읽지 않는다. 빌드에는 생성된 ScriptableObject만 들어간다.
- 가져올 때 자동 검증하고 하나라도 틀리면 생성을 중단한다.
  - 등급 확률 합계가 100%인가
  - id 중복이 없는가, 참조하는 id(스킬, 적, 프리팹)가 실제로 있는가
  - 등급별 종수가 20 / 15 / 10 / 5인가, 포지션별 종수가 기획과 같은가
  - 음수나 빈 칸이 없는가
- 서버용 값(Gacha, Shop, 보상량)은 같은 가져오기 단계에서 JSON으로도 내보내 UGS Remote Config에 올린다.
- 시트 접근: 처음에는 "링크가 있는 사용자 보기"로 공유하고 CSV 내보내기 주소로 받는다. 링크를 아는 사람은 수치를 볼 수 있으므로, 비공개가 필요해지면 Google 서비스 계정 방식으로 바꾼다.
- Claude가 시트를 직접 고치려면 Google Sheets 연동이 따로 필요하다. 당장은 Claude가 CSV를 만들어 주면 시트에 붙여 넣는 방식으로 충분하다.
- 수치를 바꿀 때는 시트의 버전 기록에 이름을 붙여 둔다(예: "v0.3 2해역 난이도 하향").

## 7. 서버 (Unity Gaming Services)

**왜 서버가 필요한가:** 법이 요구하는 것은 확률의 정확한 표기이고 서버 자체는 의무가 아니다. 하지만 인앱결제가 있는 게임에서 보석과 영입을 기기 안에서만 처리하면 메모리 변조 도구로 쉽게 조작된다. 또 "표기한 확률대로 뽑았다"는 것을 서버 기록으로 증명할 수 있어야 문의나 분쟁에 대응할 수 있다.

**영입 1회의 흐름**

```
기기: "10회 영입" 요청 → Cloud Code: 보석 확인·차감 → 확률표로 추첨 → 천장 계산 → 보유 선원·조각 저장 → 기록 남김 → 결과 반환 → 기기: 연출과 카드 표시
```

**플레이어 데이터**

| 데이터 | 쓰기 권한 |
|---|---|
| 보석, 골드, 보유 선원, 조각, 천장 횟수, 결제 기록 | 서버만 |
| 스테이지 진행, 별 | 서버(전투 결과 검사 후) |
| 편성, 설정 | 기기 |

**개발 순서**

1. 프로토타입(1단계): 서버 없이 `IGachaService`, `IWalletService`, `ISaveService` 인터페이스의 로컬 구현으로 진행한다.
2. 핵심 순환(2단계): UGS 프로젝트 생성 → 익명 로그인 → Cloud Code로 영입과 재화 처리 → 인터페이스의 서버 구현으로 교체.
3. 출시 준비(4단계): 결제 영수증 확인, 광고 보상 지급, Google Play 게임 계정 연동(기기 변경 대비).

처음부터 인터페이스로 나눠 두면 나중에 서버로 바꿀 때 화면과 전투 코드를 고치지 않아도 된다.

**비용:** UGS는 서비스마다 무료 한도가 있고 넘으면 과금된다. 한도 수치는 Unity 요금 페이지에서 확인한다(확인 필요). 호출은 영입, 보상 수령, 저장 때만 일어나도록 설계해 요청 수를 줄인다.

## 8. Tripo (3D 에셋)

**연결 가능 여부:** 가능하다(확인 필요). Tripo는 API를 제공하고, 이를 Claude Code에 붙이는 MCP 서버가 여러 개 있다.

| 후보 | 특징 |
|---|---|
| Tripo 자체 CLI의 MCP 모드 | 생성, 리깅, 애니메이션, 폴리곤 정리, 형식 변환 도구를 제공한다고 안내됨 |
| `VAST-AI-Research/tripo-mcp` | Tripo 개발사가 올린 것. 알파 단계이고 Blender 연동 중심 |
| 그 밖의 커뮤니티 서버 | 비공식. API 키를 넘기므로 코드를 확인하고 사용 |

연결하면 Claude에게 "이 콘셉트 그림으로 모델 만들고 리깅해서 FBX로 받아 줘"처럼 요청할 수 있다. Unity MCP까지 연결되어 있으면 가져오기와 프리팹 생성까지 이어서 할 수 있다.

**제작 흐름(선원 1종)**

1. 콘셉트 그림: 정면 전신, 팔을 벌린 자세, 단색 배경
2. Tripo에서 그림 → 3D 생성
3. 폴리곤 감축(목표 삼각형 5,000개 안팎), 텍스처 512×512
4. 자동 리깅 → FBX로 내려받기
5. Unity 가져오기: Rig를 Humanoid로 설정, 공용 애니메이션 연결
6. 프리팹 생성, 무기 붙이기
7. 카드 그림 렌더링(같은 구도와 조명)

**주의**

- 생성과 리깅마다 크레딧이 든다. 다시 생성하는 횟수까지 감안해 50종 예산을 잡는다.
- API 키는 환경 변수에 두고 저장소에 넣지 않는다.
- 생성물의 상업 이용 조건은 요금제마다 다를 수 있다. 약관을 확인한다(확인 필요).
- AI 생성 모델은 손가락, 얼굴, 얇은 장식에서 자주 깨진다. 탑다운 시점에서는 작게 보이므로 크게 문제 되지 않지만 선원 상세 화면에서는 가까이 보인다.
- 마스코트 5종은 사람 뼈대를 쓸 수 없다. 동물용 리깅과 동작을 따로 준비해야 한다.
- 50종의 그림체가 섞이지 않도록 콘셉트 그림 단계에서 화풍, 등신, 색 수를 고정한다.

**먼저 할 일:** 선원 1종(예: 칼잡이 준)을 1번부터 7번까지 끝내고 실제 기기에서 돌려 본다. 걸린 시간과 크레딧을 기록해 50종 일정과 예산을 계산한다.

## 9. UI 제작

- 와이어프레임 9장은 `Docs/images/`에 있다. 배치만 정한 회색 박스 초안이다.
- 순서: 와이어프레임 확정 → 회색 박스 그대로 Unity에 구현해 화면 이동과 조작 확인 → UI 킷으로 스킨 적용 → 아이콘과 카드 테두리 제작.
- 스킨을 먼저 입히면 배치를 바꿀 때마다 그림을 다시 만들어야 한다. 1~2단계는 회색 박스로 진행한다.
- Canvas Scaler: Scale With Screen Size, 기준 1080×1920. 화면 비율이 긴 기기(20:9 이상)와 노치를 안전 영역 스크립트로 처리한다.
- 선원 상세의 3D 표시는 별도 카메라로 RenderTexture에 그려 UI 이미지에 띄운다.
- 화면은 프리팹 단위로 만들고 한 번에 하나만 켠다. 공통 요소(상단 재화, 하단 탭)는 따로 둔다.

## 10. 첫 주 순서

| # | 할 일 | 담당 |
|---|---|---|
| 1 | `git status` 확인 후 첫 커밋, 비공개 원격 저장소 연결 | 사용자 승인 후 Claude |
| 2 | Notion MCP 연결, `Docs/` 문서 업로드 | 연결은 사용자, 업로드는 Claude |
| 3 | `PirateKing_Data` 시트 생성, 탭과 열 만들기, 50종 명단 입력 | CSV는 Claude, 시트 생성은 사용자 |
| 4 | Unity 빌드 설정, 폴더 구조, 씬 3개, 불필요 패키지 제거 | Claude |
| 5 | Unity MCP 연결 방식 확인 후 설치 | 사용자 + Claude |
| 6 | Tripo API 키 발급, MCP 연결, 선원 1종 시험 제작 | 사용자 + Claude |
| 7 | 전투 프로토타입 시작(선장 이동, 자동 공격, 적 1종) | Claude |

계정이 필요한 것: Notion, Google, Unity(UGS 대시보드), Tripo(API 키), Google Play Console(등록비 있음, 4단계 전까지).
