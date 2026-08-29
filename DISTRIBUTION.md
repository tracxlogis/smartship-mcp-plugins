# 배포 저장소 안내

이 저장소는 **셀러 공개 배포본**이다. 정본(단일 진실 공급원)은 `tx-front-react` 저장소의
`mcp-marketplace/` 디렉터리이며, 스킬·매니페스트 수정은 그쪽에서 하고 이 저장소와
사내 GitLab 미러(`front/smartship-mcp-plugins`)로 동기화한다 — MCP 도구 레지스트리 변경과
같은 커밋으로 스킬을 갱신하는 규칙을 지키기 위함이다.

## 사용자 등록

```
# Codex CLI
codex plugin marketplace add tracxlogis/smartship-mcp-plugins
codex plugin install smartship@tracx-mcp-marketplace

# Claude Code
/plugin marketplace add tracxlogis/smartship-mcp-plugins
/plugin install smartship@tracx-mcp-marketplace
```

설치하면 SmartShip MCP 서버가 자동 등록되고 첫 사용 때 OAuth 로그인(SmartShip 계정)이 열린다.
자세한 사용법은 SmartShip 의 [Tx SmartShip MCP 서비스 상세](https://smartship2.tracxlogis.com/services/smartship-mcp) 참고.
