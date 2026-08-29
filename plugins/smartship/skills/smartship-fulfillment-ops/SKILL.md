---
name: smartship-fulfillment-ops
description: SmartShip 3PL 풀필먼트 운영 — 재고 입고 신청(카고 생성·수정·발송 처리·취소), 풀필먼트 주문 출하지시(Pick&Pack) 요청/취소, 파손 재고 폐기/반송, 재고 반출(RTV) 신청과 담당자 메시지. "입고 신청해줘", "출하지시 걸어줘", "파손 재고 처리", "재고 반출(RTV)", inbound request, pick and pack, damaged sku, RTV 요청에 사용.
---

# SmartShip 3PL 풀필먼트 운영

풀필먼트(TxFC) 셀러의 창고 운영 업무를 대화로 수행한다: **입고 → 출하지시 → 파손/반출 처리**.
쓰기 도구는 전부 `confirm=true` — 대상을 요약 보고하고 동의받은 뒤 실행한다.

## 1. 입고 (Inbound)

- 현황: `get_inbound_request_counts`(상태 대시보드) → `list_inbound_requests`(목록) →
  `get_cargo_detail`(카고 상세)·`get_cargo_tracking_list`(추적)
- 보조: `list_inbound_location_cells`(입고 로케이션), `list_inbound_vendors`/`save_inbound_vendor`(공급처)
- 신청 흐름(**2단계 — 생성만으로는 창고에 통보되지 않는다**):
  1. `create_inbound_request` — 카고 단위 입고 신청 생성
  2. 내용 확인·수정(`update_inbound_request`, 메모는 `update_inbound_memo`)
  3. `ship_inbound_request` — **발송 처리(C2→C3)**. 이때부터 창고 접수 대상이 된다
  - 취소: `cancel_inbound_request`
- 신청 전 SKU가 등록돼 있어야 한다 — 없으면 smartship-sku-stock 스킬의 등록 절차로 먼저 안내.

## 2. 출하지시 (Pick & Pack)

- 대상 확인: smartship-orders 스킬의 `search_orders`(FF 주문) → 품절 확인 `get_out_of_stock_skus`
- `request_pick_n_pack` — 풀필먼트 주문 출하지시 요청 / `cancel_pick_n_pack` — 취소
- 출하지시 후 진행 상태는 주문 대시보드(`get_order_count_stats`)의 FF 단계로 확인한다.

## 3. 파손 재고 (Damaged SKU)

- `list_damaged_skus` — 센터가 분류한 파손 재고 조회(화면 기본: 상태 전체)
- 처리: `request_damaged_sku_disposal`(폐기) / `request_damaged_sku_return`(반송 — 주소는
  `list_return_addresses`에서 선택)
- **신청 후 7일 안에는 `withdraw_damaged_sku_request`로 철회 가능** — 폐기 신청 시 이 사실을 함께 안내한다.

## 4. 재고 반출 (RTV)

- `list_rtv_requests` → `get_rtv_request_skus`(신청별 SKU)
- 신청/수정: `save_rtv_request` (배송방법·배송지 포함) / 취소: `cancel_rtv_request`
- TX 담당자 커뮤니케이션: `list_rtv_messages` → `get_rtv_message_thread` → `send_rtv_message`,
  읽음 처리 `mark_rtv_message_read`. 반출 조건 문의는 이 채널로 담당자에게 남긴다.

## 규칙

- 창고 재고 확인은 smartship-sku-stock 스킬(`get_stock_status` 등)과 같은 창고(`pl_cd`) 기준으로 맞춘다.
- 폐기는 되돌릴 수 없다(7일 철회 기간 제외) — 대상 SKU·수량을 건별로 재확인받는다.
- 입고 신청이 "생성만 되고 발송 처리 전"인지 "발송 처리 완료"인지 상태를 항상 구분해 보고한다.
