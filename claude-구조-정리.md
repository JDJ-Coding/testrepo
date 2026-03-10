# Claude 제품 구조 정리 (최종 수정본 - MCP 경로 명확화)

> 작성일: 2026-03-01 / 최종 수정: 2026-03-11

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
  → **CLI와 VS Code 확장만 `~/.claude.json` 공유**
- **Claude Desktop MCP** ≠ **Claude Code MCP** → 완전 별개 파일

---

## 2. 제품/모드별 핵심 차이

| | Claude.ai 웹 | 데스크탑 채팅 | 데스크탑 Cowork | 데스크탑 Code | Claude Code CLI/VSCode |
|---|---|---|---|---|---|
| **성격** | 브라우저 채팅 | 일반 채팅 | 로컬 에이전트 | 개발 도우미 | 개발 도우미 |
| **스킬** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **MCP** | ✅ (원격 HTTP만) | ❌ | ✅ | ✅ | ✅ |
| **로컬 파일 접근** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **MCP 설정 위치** | 서버 (UI) | 없음 | `claude_desktop_config.json` | `claude_desktop_config.json` | `~/.claude.json` + `.mcp.json` |

---

## 3. MCP 설정 경로 **완전 분리 정리**

### 🟦 Claude Desktop (Cowork/Code 모드)

| 플랫폼 | MCP 설정 파일 경로 |
| --- | --- |
| **macOS** | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| **Windows** | `%APPDATA%\Claude\claude_desktop_config.json` |

```
데스크탑 앱 전체 설정 + Cowork/Code 모드 MCP 설정이 이 파일에 들어감
```

### 🟨 Claude Code (CLI + VS Code 확장)

| Scope | 설정 파일 경로 |
| --- | --- |
| **User/Local scope** | `~/.claude.json` (CLI/VSCode **공유**) |
| **Project scope** | 프로젝트 루트의 `.mcp.json` |

```
Claude Code CLI와 VS Code 확장은 **동일한 ~/.claude.json을 공유**
데스크탑의 claude_desktop_config.json과는 **완전 별개**
```

---

## 4. 제품/모드별 상세 설명

### Claude.ai 웹
- 브라우저에서 접속하는 독립 서비스
- MCP: Settings → Integrations (원격 HTTP만, 로컬 stdio 불가)
- 로컬 설정 파일 없음

### 데스크탑 채팅 모드
- Claude.ai 웹과 유사한 일반 채팅
- MCP 미지원

### 데스크탑 Cowork 모드
- 로컬 파일/앱/브라우저 접근하는 에이전트 모드
- **MCP 설정**: `claude_desktop_config.json`

### 데스크탑 Code 모드 / Claude Code CLI / VS Code 확장
- **데스크탑 Code 모드**: `claude_desktop_config.json` 사용
- **Claude Code CLI/VS Code 확장**: `~/.claude.json` 사용
- **CLI ↔ VS Code 확장만 설정 파일 공유**

---

## 5. 스킬/MCP 사용 방법

### 스킬 호출 (공통)
```
/스킬명
/xlsx 이 데이터 스프레드시트로 만들어줘
```

### MCP 사용 예시
```
"Notion 페이지 가져와줘" → Notion MCP
"React Query 문서 찾아줘" → Context7 MCP
```

### 스킬 설치 (Claude Code CLI)
```bash
npx @smithery/cli@latest skill add [스킬명] --agent claude-code
```

---

## 6. Claude Code MCP scope

```bash
# User scope (전역) → ~/.claude.json 최상위
claude mcp add --transport http --scope user notion https://mcp.notion.com/mcp

# Local scope (기본값) → ~/.claude.json의 projects 항목
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Project scope → 프로젝트 루트 .mcp.json
claude mcp add --transport http --scope project notion https://mcp.notion.com/mcp
```

---

## 7. 공유 여부 **명확 정리**

| 항목 | Claude.ai 웹 ↔ 데스크탑 | 데스크탑 ↔ Claude Code CLI/VSCode | CLI ↔ VS Code 확장 |
| --- | --- | --- | --- |
| **MCP 설정** | ❌ 별개 | ❌ **완전 별개 파일** | ✅ `~/.claude.json` 공유 |
| **스킬** | ❌ 별개 | ❌ 별개 | ✅ 공유 |

---

## 8. FAQ

**Q. Claude Desktop과 Claude Code의 MCP가 공유되나요?**  
→ **절대 아님.**  
```
Claude Desktop: claude_desktop_config.json (OS별 경로)
Claude Code: ~/.claude.json + 프로젝트 루트 .mcp.json
→ 각각 독립적으로 설정해야 함
```

**Q. VS Code 확장은 Claude Code CLI와 같은 설정을 쓰나요?**  
→ **네, `~/.claude.json`을 공유**합니다.  
데스크탑의 `claude_desktop_config.json`과는 무관.

**Q. CLAUDE.md 위치?**  
- 전역: `~/.claude/CLAUDE.md`  
- 프로젝트: `[프로젝트]/CLAUDE.md`

**Q. Cowork 마켓플레이스 에러?**  
→ `CoworkVMService` 중단. `Claude-재시작.ps1` 실행.

---

> ✅ **핵심:**  
> Claude Desktop MCP (`claude_desktop_config.json`) ≠ Claude Code MCP (`~/.claude.json`)  
> **CLI ↔ VS Code 확장만 공유**, 데스크탑은 완전 별도입니다.
