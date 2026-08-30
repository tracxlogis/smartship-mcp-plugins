# TracX SmartShip MCP Plugin

**English** | [한국어](#한국어)

Connect your TracX Logis **SmartShip** seller account to an AI assistant. Installing this plugin sets up
the SmartShip MCP connection **and** the task skills that tell the assistant how to run your daily
shipping work — orders, waybills, pickups, SKU and stock, settlement, and more.

Once installed you can simply ask: *"What do I need to handle today?"*, *"Any orders waiting for
waybills?"*, *"How much stock does this SKU have?"*

## Requirements

- A TracX Logis SmartShip seller account (you sign in during the first use).
- One of these clients: **Claude Code**, **Codex CLI**, **Claude desktop app**, or the
  **ChatGPT desktop app**.

## Install

### Claude Code

```
/plugin marketplace add tracxlogis/smartship-mcp-plugins
/plugin install smartship@tracx-mcp-marketplace
```

### Codex CLI

```
codex plugin marketplace add tracxlogis/smartship-mcp-plugins
codex plugin install smartship@tracx-mcp-marketplace
```

### Claude desktop app

1. Open **Settings → Plugins** and click **Add** in the top right.
2. Choose **Add from repository** and enter `tracxlogis/smartship-mcp-plugins`.
3. Install **SmartShip** from the plugin list of the marketplace you just added.

### ChatGPT desktop app

1. Open **Plugins** in the sidebar and click **Add** in the top right.
2. In **Add plugin marketplace**, enter `tracxlogis/smartship-mcp-plugins` as the source.
   Leave the Git ref as `main` and the sparse path empty.
3. Install **SmartShip** from the plugin list.

On first use a browser window opens so you can sign in with your SmartShip account and approve the
requested access. Menu names and layout differ slightly between app versions and languages.

## What you can ask

| Area | Example |
|---|---|
| Daily briefing | "Summarize what I need to handle today" |
| Orders | "Show my order status for the last 30 days" |
| Shipments and tracking | "Where is this tracking number right now?" |
| Waybills | "Assign couriers to these orders and create the waybill PDF" |
| Pickups | "Check if pickup is available tomorrow and request it" |
| SKU and stock | "List SKUs that are out of stock" |
| Fulfillment | "How are my inbound requests going?" |
| Claims | "Show pending claims" |
| Billing | "What's my TxMoney balance and this month's expected deduction?" |

If you do not mention a period or filter, the assistant queries with the **same default conditions as
the matching SmartShip screen**, so the numbers match what you see in the web app.

## Included skills

| Skill | Covers |
|---|---|
| `smartship-daily-briefing` | Everything waiting for you today, in priority order |
| `smartship-orders` | Order dashboard, search, detail, confirm, dispatch, hold, split |
| `smartship-shipments` | Delivery status, tracking, failed deliveries, redelivery and return |
| `smartship-waybills` | Waybill sections, courier assignment and cancellation, printing and reprinting |
| `smartship-pickups` | Pickup-ready orders, available dates, fees, request and confirm |
| `smartship-sku-stock` | SKU search and detail, warehouse stock, closing reports |
| `smartship-fulfillment-ops` | Inbound requests, pick and pack, damaged stock, return to vendor |
| `smartship-issue-clinic` | Why an order is stuck, and the fix for each cause |
| `smartship-claims-cs` | Channel claims, approval and rejection, exchange tracking, CS memos |
| `smartship-billing` | TxMoney balance and history, invoices, bank deposits, exchange rates |
| `smartship-stock-sync` | Channel stock sync settings, manual sync, "channel stock is wrong" diagnosis |
| `smartship-order-analytics` | Orders by channel, top products and SKUs, period summaries |

## How your data is handled

- The assistant only reaches **your own account's data**, through your SmartShip sign-in.
- Actions that change data — confirming orders, assigning couriers, requesting pickups — run only
  after the assistant summarizes the targets and you agree.
- For actions that cost money, such as courier assignment or pickups, the cost and your TxMoney
  balance are shown before anything runs.
- API Key connections are mostly read-only. Use the OAuth sign-in connection if you need write actions.

## Troubleshooting

- **Installed, but nothing happens.** Check that the plugin is enabled, then start a new conversation.
- **Sign-in window does not open.** Check your browser's pop-up blocker and sign in again.
- **Numbers differ from the web app.** Ask the assistant which period and filters it used; a different
  period is the usual cause.
- **A request is refused.** Some features have no tool yet and are only available on the SmartShip
  screens — the assistant will say so and point you to the right screen.

## Support

Connection guide and detailed usage: <https://smartship2.tracxlogis.com/services/smartship-mcp>

---

## 한국어

[English](#tracx-smartship-mcp-plugin) | **한국어**

TracX Logis **SmartShip** 셀러 계정을 AI 어시스턴트에 연결합니다. 이 플러그인을 설치하면 SmartShip MCP
연결과 함께, 주문·송장·픽업·SKU/재고·정산 같은 매일 하는 배송 업무를 어떻게 처리하는지 알려 주는 업무
스킬이 한 번에 설정됩니다.

설치 후에는 이렇게 물어보면 됩니다. *"오늘 처리할 일 정리해줘"*, *"송장 뽑을 주문 있어?"*,
*"이 SKU 재고 몇 개야?"*

### 사용 전제

- TracX Logis SmartShip 셀러 계정 (첫 사용 때 로그인합니다).
- 클라이언트 중 하나: **Claude Code**, **Codex CLI**, **Claude 데스크톱 앱**, **ChatGPT 데스크톱 앱**.

### 설치

#### Claude Code

```
/plugin marketplace add tracxlogis/smartship-mcp-plugins
/plugin install smartship@tracx-mcp-marketplace
```

#### Codex CLI

```
codex plugin marketplace add tracxlogis/smartship-mcp-plugins
codex plugin install smartship@tracx-mcp-marketplace
```

#### Claude 데스크톱 앱

1. **설정 → 플러그인**을 열고 오른쪽 위 **추가**를 누릅니다.
2. **저장소에서 추가**를 고르고 `tracxlogis/smartship-mcp-plugins` 를 입력합니다.
3. 추가된 마켓플레이스의 플러그인 목록에서 **SmartShip** 을 설치합니다.

#### ChatGPT 데스크톱 앱

1. 사이드바에서 **플러그인**을 열고 오른쪽 위 **추가**를 누릅니다.
2. **플러그인 마켓플레이스 추가** 창의 출처에 `tracxlogis/smartship-mcp-plugins` 를 입력합니다.
   Git ref 는 `main`(기본값), Sparse 경로는 비워 둡니다.
3. 플러그인 목록에서 **SmartShip** 을 설치합니다.

첫 사용 때 브라우저가 열리면 SmartShip 계정으로 로그인하고 요청된 접근 권한에 동의합니다. 메뉴 이름과
배치는 앱 버전·언어에 따라 조금씩 다를 수 있습니다.

### 무엇을 요청할 수 있나

| 영역 | 예시 |
|---|---|
| 업무 브리핑 | "오늘 처리할 일 정리해줘" |
| 주문 | "최근 30일 주문 현황 보여줘" |
| 배송·추적 | "이 송장번호 지금 어디까지 갔어?" |
| 송장 | "이 주문들 배송사 할당하고 송장 PDF 만들어줘" |
| 픽업 | "내일 픽업 가능한지 확인하고 신청해줘" |
| SKU·재고 | "재고 없는 SKU 목록 뽑아줘" |
| 풀필먼트 | "입고 신청 진행 상황 알려줘" |
| 클레임 | "처리 대기 중인 클레임 보여줘" |
| 정산 | "TxMoney 잔액이랑 이번 달 차감예정액 알려줘" |

기간이나 조건을 말하지 않으면 **해당 SmartShip 화면의 기본 검색 조건과 같은 기준**으로 조회하므로,
웹 화면에서 보는 숫자와 일치합니다.

### 포함된 스킬

| 스킬 | 다루는 일 |
|---|---|
| `smartship-daily-briefing` | 오늘 처리할 일을 우선순위로 정리 |
| `smartship-orders` | 주문 대시보드·검색·상세, 주문확인·출하지시·보류·분할 |
| `smartship-shipments` | 배송 현황·추적, 배송 실패, 재배송·회수 |
| `smartship-waybills` | 송장 섹션 집계, 배송사 할당·취소, 출력·재출력 |
| `smartship-pickups` | 픽업 대기 주문, 가능일·픽업비, 신청과 확정 |
| `smartship-sku-stock` | SKU 검색·상세, 창고별 재고, 마감 리포트 |
| `smartship-fulfillment-ops` | 입고 신청, 출하지시, 파손 재고, 재고 반출(RTV) |
| `smartship-issue-clinic` | 주문이 막힌 원인 진단과 원인별 조치 |
| `smartship-claims-cs` | 채널 클레임 승인·거절, 교환 송장 등록, CS 메모 |
| `smartship-billing` | TxMoney 잔액·내역, 인보이스, 무통장 입금, 환율 |
| `smartship-stock-sync` | 채널 재고 동기화 설정·수동 동기화, "채널 재고가 안 맞아요" 진단 |
| `smartship-order-analytics` | 채널별 주문, 상품·SKU 판매 순위, 기간 요약 |

### 데이터는 이렇게 다뤄집니다

- 어시스턴트는 로그인한 **본인 계정의 데이터에만** 접근합니다.
- 주문확인·배송사 할당·픽업 신청처럼 데이터를 바꾸는 작업은, 대상과 내용을 먼저 요약해 보여주고
  동의한 뒤에만 실행됩니다.
- 배송사 할당·픽업처럼 비용이 발생하는 작업은 실행 전에 비용과 TxMoney 잔액을 알려 줍니다.
- API Key 방식으로 연결하면 대부분 조회만 가능합니다. 처리 작업까지 쓰려면 OAuth 로그인 방식으로
  연결하세요.

### 문제 해결

- **설치했는데 아무 반응이 없습니다.** 플러그인이 활성화돼 있는지 확인하고 새 대화를 시작합니다.
- **로그인 창이 열리지 않습니다.** 브라우저의 팝업 차단을 확인하고 다시 시도합니다.
- **웹 화면과 숫자가 다릅니다.** 어떤 기간·조건으로 조회했는지 물어보세요. 기간 차이가 가장 흔한
  원인입니다.
- **요청이 거절됩니다.** 아직 도구가 없어 화면에서만 되는 기능이 있습니다. 이 경우 어시스턴트가
  해당 화면을 안내합니다.

### 문의

연결 가이드와 자세한 사용법: <https://smartship2.tracxlogis.com/services/smartship-mcp>
