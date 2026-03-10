# Claude 제품 구조 정리 (수정안)

> 작성일: 2026-03-01 / 최종 수정: 2026-03-10


---

## 1. 전체 구조

```
┌──────────────┬──────────────────────────────────┬────────────────────────────────────┐
│ Claude.ai 웹 │       Claude 데스크탑 앱           │         Claude Code                │
│  (브라우저)  │          (Electron 앱)             │                                  │
│              │  ┌──────┬──────────┬──────────┐   │  ┌─────────────┬────────────────┐  │
│              │  │채팅  │Cowork 모드│Code 모드 │   │  │ CLI (터미널) │ VS Code 확장   │  │
│              │  └──────┴──────────┴──────────┘   │  └─────────────┴────────────────┘  │
└──────────────┴──────────────────────────────────┴────────────────────────────────────┘
```

**핵심 관계:**
- **Claude.ai 웹** = 브라우저에서 접속하는 독립 서비스 (설치 필요 없음, 서버 설정)
- **데스크탑 채팅 모드** ≈ Claude.ai 웹 (Anthropic 서버 연결, 유사한 기능)
- **데스크탑 Cowork 모드** → 로컬 파일/앱/브라우저에 직접 접근하는 에이전트 모드
- **데스크탑 Code 모드** ↔ **Claude Code CLI** ↔ **Claude Code VS Code 확장**  
  → **세 가지 모두 동일한 설정 파일 구조** 사용 (`~\.claude.json`, `~\.claude\`)
- **Claude Code CLI / VS Code 확장**은 **동일한 `~\.claude.json` 파일을 공유**


---

## 2. 제품/모드별 핵심 차이

| | Claude.ai 웹 | 데스크탑 채팅 | 데스크탑 Cowork | 데스크탑 Code | Claude Code CLI / VSCode |
|---|---|---|---|---|---|
| **성격** | 브라우저 채팅 | 일반 채팅 | 로컬 에이전트 | 개발 도우미 | 개발 도우미 |
| **스킬** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **MCP** | ✅ (원격 HTTP MCP만) | ❌ | ✅ | ✅ | ✅ |
| **로컬 파일 접근** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **설정 저장 위치** | 서버 (파일 없음) | 서버 (파일 없음) | `AppData\Roaming\Claude\` | `~\.claude\` | `~\.claude\` |
| **MCP 설정 위치** | 서버 (UI) | 없음 | `claude_desktop_config.json` | `~\.claude.json` | `~\.claude.json` |


---

## 3. 제품/모드별 상세 설명


### Claude.ai 웹

- 브라우저에서 접속하는 독립 서비스 (설치 불필요)
- 스킬 사용 가능 (`/스킬명` 호출)
- **MCP 지원**: Settings → Integrations에서 원격 HTTP MCP 서버 연결 가능 (서버 측 저장)
- 로컬 파일 접근 불가, 로컬에 설정 파일 없음  
- **주의**: 로컬 stdio 방식 MCP는 지원하지 않음


### 데스크탑 채팅 모드

- Claude.ai 웹과 유사한 일반 채팅
- 스킬 사용 가능
- MCP·로컬 파일 접근 불가
- 로컬 설정 파일 없음 (설정은 서버와 `AppData\Roaming\Claude\claude_desktop_config.json` 등 공통 앱 설정만 존재)


### 데스크탑 Cowork 모드

- 로컬 파일, 앱, 브라우저에 직접 접근하는 에이전트 모드
- 스킬은 세션 시작 시 자동 주입
- MCP 지원 (`claude_desktop_config.json`에 설정)

**데이터 경로:**
```
%APPDATA%\Claude\                           (= C:\Users\[사용자]\AppData\Roaming\Claude\)
├── claude_desktop_config.json             ← MCP 서버 설정 + 앱 전체 설정
├── local-agent-mode-sessions\             ← Cowork 세션/스킬/플러그인
└── logs\                                  ← 앱 로그
```


### 데스크탑 Code 모드 / Claude Code CLI / VS Code 확장

- 코드 작업에 특화된 개발 도우미
- **세 가지 모두 동일한 설정 구조** 사용:
  - `~\.claude.json` (전역 설정, MCP 등)
  - `~\.claude\` 아래 권한/스킬/플러그인/프로젝트별 메모리
- 스킬은 `/스킬명` 슬래시 명령어로 호출
- MCP 지원 (`~\.claude.json`에 설정)

**데이터 경로:**
```
~\                                          (= C:\Users\[사용자]\)
└── .claude.json                          ← MCP 서버 등록, 프로젝트별 설정 등 전체 설정


~\.claude\                                (= C:\Users\[사용자]\.claude\)
├── settings.json                         ← bash 명령 허용/거부 권한 설정 (전역)
├── settings.local.json                   ← 로컬 권한 설정 (프로젝트별 override)
├── skills\                               ← 설치된 스킬
│   └── [스킬명]\                         ← 스킬별 디렉토리
├── plugins\                              ← 플러그인 캐시/차단목록
└── projects\                             ← 프로젝트별 메모리
    └── [경로]\memory\MEMORY.md
```

> **`~\.claude.json` vs `~\.claude\settings.json` 차이:**
> - `~\.claude.json` → MCP 서버 등록, 프로젝트별 허용 도구 등 **Claude Code 전체 설정**
> - `~\.claude\settings.json` → **bash 명령 허용/거부 권한 목록** (보안 설정, 전역)

**현재 설치된 스킬 (`~\.claude\skills\` 기준):**
| 스킬 | 설명 |
|---|---|
| anthropics-skill-creator | 스킬 생성/관리 |
| xlsx | 스프레드시트 작업 |


---

## 4. 스킬/MCP 사용 방법


### 스킬 호출

```
# Claude.ai 웹 / 데스크탑 채팅 / 데스크탑 Code / CLI / VS Code 확장 공통
/스킬명
/xlsx 이 데이터 스프레드시트로 만들어줘
```

**Cowork 모드:**
- 세션 시작 시 자동으로 스킬 주입, 별도 슬래시 명령 불필요  
- 추가로 `/스킬명` 으로 호출 가능하지만, 기본적으로 **세션별 자동 주입**이 핵심 특징


### MCP 사용 (Claude Code / 데스크탑 Cowork)

```
# 자연어로 요청하면 자동으로 해당 MCP 서버 사용
"Notion 페이지 가져와줘"           → Notion MCP
"React Query v5 문서 찾아줘"       → Context7 MCP
```

> **추가 공지:**
- Claude.ai 웹 MCP는 **원격 HTTP 서버만**, 로컬 stdio 방식은 지원하지 않음


### 스킬 설치 방법 (Claude Code CLI)

```bash
# Smithery CLI로 설치
npx @smithery/cli@latest skill add [스킬명] --agent claude-code

# 예시
npx @smithery/cli@latest skill add anthropics/data-exploration --agent claude-code
```


---

## 5. MCP 설정 경로 정리

| 앱/모드 | MCP 설정 파일 경로 | MCP 추가 방법 |
|---|---|---|
| **Claude.ai 웹** | 파일 없음 (서버 저장) | Settings → Integrations (UI) |
| **데스크탑 채팅 모드** | 없음 (MCP 미지원) | - |
| **데스크탑 Cowork / Code (데스크탑 앱)** | `%APPDATA%\Claude\claude_desktop_config.json` | 파일 직접 편집 |
| **Claude Code CLI** | `~\.claude.json` | `claude mcp add --scope user ...` |
| **Claude Code VS Code 확장** | `~\.claude.json` (CLI와 동일) | CLI와 동일 |
| **VS Code 자체 MCP** *(Claude 아님)* | `.vscode/mcp.json` | 파일 직접 편집 |

> **헷갈리는 포인트:**
> - Claude Code CLI와 VS Code 확장은 **같은 `~\.claude.json` 파일을 씀**
> - "VS Code MCP"는 VS Code 1.99에서 추가된 **에디터 자체 기능** → Claude Code와 무관
> - Claude Desktop은 현재 이 PC엔 **설치 안 됨** (경로 없음)
> - **Claude.ai 웹 MCP는 원격 HTTP 방식만 가능**. 로컬 stdio 방식은 지원하지 않음

### Claude Code MCP scope 종류

```bash
# user scope — 모든 프로젝트에서 사용 가능 (전역), ~/.claude.json 최상위에 저장
claude mcp add --transport http --scope user notion https://mcp.notion.com/mcp

# local scope (기본값) — 현재 프로젝트만, ~/.claude.json 의 projects 항목에 저장
claude mcp add --transport http notion https://mcp.notion.com/mcp

# project scope — 팀 공유용, 프로젝트 루트의 .mcp.json 에 저장
claude mcp add --transport http --scope project notion https://mcp.notion.com/mcp
```

> **scope 정리:**
> - `user` → 당신만 모든 프로젝트에서 사용
> - `local` → 현재 프로젝트만 사용 (프로젝트 단위 개인 설정)
> - `project` → `.mcp.json` 로 커밋, 팀 공유


---

## 6. 공유 여부

| 항목 | Claude.ai 웹 ↔ 데스크탑 채팅 | 데스크탑 ↔ Claude Code | Claude Code CLI ↔ VS Code 확장 |
|---|---|---|---|
| 스킬 | ❌ 별개 | ❌ 별개 | ✅ 공유 |
| MCP | ❌ 별개 | ❌ 별개 | ✅ 공유 |
| 설정 파일 | ❌ 별개 | ❌ 별개 | ✅ 공유 |

> ✅ **정리:**
> - **Claude Code CLI와 VS Code 확장은 동일한 설정 파일을 공유**하므로, 여기서는 **공유(O)**로 표기하는 것이 맞음
> - **데스크탑 Cowork / Code 모드**는 **별개 설정 파일**을 사용하므로, 데스크탑 ↔ Claude Code는 **공유 안 함(X)**


---

## 7. FAQ

**Q. Cowork 마켓플레이스가 "불러오지 못했습니다" 에러가 나요**  
→ `CoworkVMService`가 중단된 것이 의심됨. 바탕화면의 `Claude-재시작.ps1` 실행 권장


**Q. settings 파일이 여러 개인가요?**  
→  
- `~\.claude.json` : Claude Code 전체 설정 (**MCP 서버, 프로젝트별 허용 도구 등**)  
- `~\.claude\settings.json` : **bash 명령 허용/거부 권한 설정** (전역)  
- `%APPDATA%\Claude\claude_desktop_config.json` : Claude Desktop 앱 설정 (MCP 포함)  

> **추가:**
> - `.<프로젝트>/.claude/settings.json` : 프로젝트 공유 설정 (팀 공유)  
> - `.<프로젝트>/.claude/settings.local.json` : 프로젝트별 개인 설정


**Q. CLAUDE.md는 어디에 두나요?**  
→  
- 전역 적용: `~\.claude\CLAUDE.md`  
- 프로젝트별: `[프로젝트 폴더]\CLAUDE.md`


**Q. Claude.ai 웹에서 MCP를 쓰려면?**  
→  
- claude.ai → Settings → Integrations → MCP 서버 URL 입력  
- **원격 HTTP 방식만 가능**. 로컬 stdio 방식(파이썬 스크립트 등)은 지원하지 않음


**Q. Claude Code와 Claude Desktop의 MCP가 공유되나요?**  
→ **아니요.**  
- Claude Code: `~\.claude.json`  
- Claude Desktop: `%APPDATA%\Claude\claude_desktop_config.json`  
→ 서로 다른 설정 파일이라, 각각 **별도로 등록**해야 함