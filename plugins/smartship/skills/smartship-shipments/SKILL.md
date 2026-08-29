---
name: smartship-shipments
description: SmartShip 배송 현황·추적·배송실패 — 배송중/완료/실패/반송 대시보드 집계, 배송 목록·일별 건수, 송장번호 트래킹, 통관보류·NDR 재배송(RED)/회수(RTO)·장기보관 처리. "배송 현황 알려줘", "이 송장 어디쯤이야", "배송 실패 건 처리", shipment status, tracking, failed delivery, redelivery, RTO 요청에 사용.
---

# SmartShip 배송 현황·추적

배송 현황 화면과 배송 실패(Exceptions) 화면의 업무를 대화로 수행한다. 순서:
**① 상태 대시보드 집계 → ② 상태별 목록 → ③ 건별 추적/상세 → ④ (요청 시) 실패 건 후속 처리**.

## 화면 기본 조회 조건 (사용자가 조건을 말하지 않으면 이 값을 쓴다)

- **배송 현황**(`search_shipments`·`get_shipment_status_summary`·`get_shipment_daily_counts`):
  조회일 기준 `REG_DT`(주문 등록일) · 기간 **최근 15일**(한 번에 최대 31일) ·
  검색 타입 `SHIPPING_NO`(QSP) · 창고/등록 방식/출고 유형/국가 필터 전체(빈값)
- **배송 실패**(`get_failed_delivery_counts`·`search_failed_delivery_orders`):
  배송 실패 등록일 기준 **최근 30일**(한 번에 최대 90일) · 검색 타입 `BN`(QSP) · 출고 유형 전체
- 집계와 목록은 같은 기간으로 조회한다 — 화면과 숫자가 다르면 기간·기간 기준부터 대조한다.

## 1. 현황 파악 (대시보드)

- `get_shipment_status_summary` — 배송중/배송완료/배송실패/반송 상태별·세부 단계별 건수.
- `get_shipment_daily_counts` — 현재 조건의 일자별 배송 건수(추이 질문에 사용).
- `get_failed_delivery_counts` — 통관보류·배송사 처리불가·장기보관/반송·재배송·폐기·재고원복 건수.

## 2. 목록 조회

- `search_shipments` — 상태 코드+검색 조건으로 배송 주문 목록.
- `search_failed_delivery_orders` — 실패 유형·사유 코드 기준 배송실패 주문 목록.
- `get_long_keep_due_orders` — 폐기 임박 장기보관 주문(건수+목록). "곧 폐기되는 건 있어?" 질문에 사용.
- 필터 후보값: `list_shipment_sales_channels`(판매사이트/샵), `list_shipment_warehouses`(창고/DPC).

## 3. 건별 추적·상세

- `get_tracking_history` — 송장번호로 배송 이력 타임라인·최신 상태(최대 30건).
- `get_exception_order_items` — 실패 건 후속 처리 전 선택 주문의 상품 라인 확인.

## 4. 실패 건 후속 처리 (쓰기 — 반드시 아래 절차)

쓰기 도구는 `confirm=true` 필수. **대상 건과 처리 방식(재배송/회수/재고원복/폐기)을 요약해 사용자
동의를 받은 뒤** 실행한다. 특히 폐기(DP)는 되돌릴 수 없으므로 건별로 재확인한다.

- `get_ndr_redelivery_targets` — NDR 재배송/RTO 요청 전 실제 처리 대상 확인(먼저 호출).
- `request_redelivery_or_rto` — NDR 주문 재배송(RED) 또는 회수/RTO 요청.
- `request_lk_restock_or_discard` — 장기보관(LK) 주문의 재고원복(RS) 또는 폐기(DP) 신청.

## 규칙

- "배송이 왜 안 가요?" 류 질문은 ① 주문 상태(orders 스킬의 `get_order_detail`) ② 배송 추적
  (`get_tracking_history`) ③ 실패 목록 순으로 원인을 좁혀 보고한다.
- 집계와 목록 건수가 다르면 조건(기간·창고·채널)이 다른 것이다 — 같은 조건으로 맞춰 다시 확인한다.
- 송장 출력·배송사 할당·재출력은 현재 도구가 없다. 송장 출력 화면(/waybills) 이용을 안내한다.
