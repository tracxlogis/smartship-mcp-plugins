# 배포 저장소 안내

이 저장소는 **배포본**이다. 정본(단일 진실 공급원)은 `tx-front-react` 저장소의
`mcp-marketplace/` 디렉터리이며, 스킬·매니페스트 수정은 그쪽에서 하고 이 저장소로
동기화한다 — MCP 도구 레지스트리(`src/lib/mcp/registry.ts`) 변경과 같은 커밋으로
스킬을 갱신하는 규칙을 지키기 위함이다.

## 사용자 등록

```
# Codex CLI
codex plugin marketplace add https://gitlab.qxpress.net/front/smartship-mcp-plugins.git
codex plugin install smartship@tracx-mcp-marketplace

# Claude Code
/plugin marketplace add https://gitlab.qxpress.net/front/smartship-mcp-plugins.git
/plugin install smartship@tracx-mcp-marketplace
```

설치하면 SmartShip MCP 서버가 자동 등록되고 첫 사용 때 OAuth 로그인(SmartShip 계정)이 열린다.
