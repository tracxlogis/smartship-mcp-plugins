# TracX SmartShip MCP 플러그인 마켓플레이스

SmartShip MCP 서버(`https://smartship2.tracxlogis.com/api/mcp`)를 **플러그인 1개(연결 설정 + 업무 스킬)로 패키징**해,
사용자가 카탈로그 한 번 등록으로 설치·업데이트할 수 있게 하는 배포 디렉터리다.

> 마켓플레이스 = OpenAI가 운영하는 스토어가 아니라 **우리가 git에 올려두는 플러그인 카탈로그(JSON 매니페스트)**.
> 사용자는 `plugin marketplace add <저장소>` 로 이 카탈로그를 소스로 추가하고, 그 안의 플러그인을 골라 설치한다.
> Codex CLI(`.agents/plugins/marketplace.json`)와 Claude Code(`.claude-plugin/marketplace.json`) 두 형식을 함께 제공한다.

## 구조

```
mcp-marketplace/                    ← 마켓플레이스 루트 (배포 시 이 디렉터리가 저장소 루트가 된다)
  .agents/plugins/marketplace.json  ← Codex 카탈로그
  .claude-plugin/marketplace.json   ← Claude Code 카탈로그
  plugins/
    smartship/                      ← 플러그인: MCP 연결 + 업무 스킬
      .codex-plugin/plugin.json     ← Codex 매니페스트
      .claude-plugin/plugin.json    ← Claude Code 매니페스트
      .mcp.json                     ← SmartShip 원격 MCP 서버 연결 설정 (OAuth 2.1 자동 탐색)
      skills/
        smartship-orders/SKILL.md      ← 주문 관리 (/orders/new)
        smartship-shipments/SKILL.md   ← 배송 현황·추적·실패 (/orders/shipments, /tracking, /orders/exceptions)
        smartship-pickups/SKILL.md     ← 픽업 관리 (/pickups)
        smartship-waybills/SKILL.md    ← 송장 출력·배송사 할당 (/waybills, /waybills/history)
        smartship-sku-stock/SKILL.md   ← SKU·재고 (/inventory/skus, /stock, /reports)
        smartship-daily-briefing/SKILL.md   ← 오늘의 업무 브리핑 (도메인 횡단 집계)
        smartship-order-analytics/SKILL.md  ← 주문 요약·분석 (채널/상품·SKU/국가/거래 요약)
        smartship-fulfillment-ops/SKILL.md  ← 3PL 풀필먼트 운영 (입고·출하지시·파손·RTV)
        smartship-issue-clinic/SKILL.md     ← 문제 주문 해결 (에러/보류 진단→유형별 조치)
        smartship-claims-cs/SKILL.md        ← 클레임·배송 CS (/customer-service/delivery)
        smartship-billing/SKILL.md          ← 정산·인보이스·TxMoney (/billing/tx-money, /invoice)
        smartship-stock-sync/SKILL.md       ← 재고 동기화 (/stock/sync-settings, /stock/sync-history)
```

## 설계 원칙 (스킬)

1. **페이지 단위·사용자 관점** — 스킬 1개 = 셀러가 실제로 쓰는 주요 화면 1~2개의 업무 흐름.
   화면의 일반적인 작업 방식(**대시보드 집계 → 조건 좁혀 목록 → 건별 상세 → 필요 시 처리**)을 그대로 순서로 옮긴다.
2. **실존 도구만 참조** — 스킬 본문의 도구 이름은 `src/lib/mcp/registry.ts` 에 실제 등록된 이름만 쓴다.
   registry 의 도구를 추가·개명하면 해당 스킬도 같은 변경에서 함께 갱신한다.
3. **쓰기 도구 안전 수칙 공통화** — `*:write` scope 도구는 전부 `confirm=true` 필수. 스킬은
   "대상 확정 → 사용자에게 요약 보고 → 명시 동의 후 실행" 순서를 강제한다. 인증 컨텍스트(cust_no 등)는
   서버가 주입하므로 스킬이 입력받지 않는다.
4. **내부 구현 비노출** — 스킬 본문에 SP 이름·레거시 경로·서버 내부 구조를 쓰지 않는다
   (사용자 대상 문서 원칙, `docs/manual/` 과 동일 기준).

## 사용자 등록 방법

**셀러 공개 배포 저장소: https://github.com/tracxlogis/smartship-mcp-plugins** —
이 디렉터리가 정본이고 위 GitHub 저장소로만 동기화한다(사내 GitLab 미러는 이중 유지
불필요로 2026-08-29 폐기).

### Codex CLI

```
codex plugin marketplace add tracxlogis/smartship-mcp-plugins
codex plugin install smartship@tracx-mcp-marketplace
```

### Claude Code

```
/plugin marketplace add tracxlogis/smartship-mcp-plugins
/plugin install smartship@tracx-mcp-marketplace
```

설치하면 `.mcp.json` 의 SmartShip 서버가 자동 등록되고 첫 호출 때 OAuth 2.1 로그인(SmartShip 계정)이 열린다.
API Key 인증 등 다른 연결 방법은 `/services/smartship-mcp` 페이지 참고.

### ChatGPT 웹/데스크톱

ChatGPT 웹의 일반 사용자는 마켓플레이스 대신 **개발자 모드 커넥터**로 MCP URL 을 직접 추가한다
(`/services/smartship-mcp` 의 ChatGPT 앱 절차). 이 카탈로그는 Codex CLI·Claude Code 사용자용이며,
ChatGPT 공개 플러그인 디렉토리 제출은 아래 실행 계획 Phase 4 의 별도 결정 사항이다.

## 실행 계획

| Phase | 내용 | 상태 |
|---|---|---|
| 0 | MCP 서버 운영(도구 약 160종, 17그룹) + `/services/smartship-mcp` 연결 가이드 | 완료(선행) |
| 1 | **주요 페이지 스킬 + 플러그인 + 마켓플레이스 스캐폴드** (이 디렉터리) | 완료 |
| 2 | 송장 출력(`/waybills`) MCP 도구 CRUD 12종(`src/lib/mcp/waybill-tools.ts`, group `waybills`) + `smartship-waybills` 스킬. 미포함: 인도 Momoe 라벨(A6/A4)·패킹 라벨 출력 | 완료 |
| 3 | 셀러 워크플로 스킬 5종(브리핑·주문 분석·풀필먼트 운영·문제 주문 해결·클레임 CS) + TxMoney 조회 도구 4종(`get_txmoney_accounts`/`get_txmoney_monthly_deduction`/`search_txmoney_history`/`get_txmoney_history_count`, billing 그룹) | 완료 |
| 3b | 정산·인보이스(billing)·재고 동기화(stock-sync) 스킬 + 무통장 입금 도구 2종(`get_txmoney_bank_deposits`/`notify_bank_deposit_paid`, `billing:write` scope 신설). 장기 잔여: SKU별 판매 집계 SP(서버측 집계, DB 작업+Redmine 검수), 인도 Momoe 라벨 도구(제외 결정 2026-08-29) | 완료 |
| 4 | 배포: 셀러 공개 GitHub 저장소 `tracxlogis/smartship-mcp-plugins` 신설(단일 배포 채널 — 사내 GitLab 미러는 폐기), `/services/smartship-mcp` 개발자 클라이언트 절에 플러그인 설치 절차 추가. ChatGPT 공개 디렉토리 제출 여부는 별도 결정(심사용 테스트 셀러 계정 필요) | 완료 |

## 유지보수

- 도구 추가/변경(`src/lib/mcp/registry.ts`) 시 관련 스킬 본문·`docs/mcp/page-index.md` 를 함께 갱신한다.
- 스킬 문구는 사용자 대상이므로 `.cursor/rules/user-facing-message-content.mdc` 의 청자 구분 원칙을 따른다.
- 버전은 플러그인 `plugin.json` 의 `version` 으로 관리하고, 카탈로그 등록 사용자는 재설치 없이 업데이트를 받는다.
