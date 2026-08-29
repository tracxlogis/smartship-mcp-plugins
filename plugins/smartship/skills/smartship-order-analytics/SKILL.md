---
name: smartship-order-analytics
description: SmartShip 주문 요약·분석 — 판매채널별 주문 비중, 상품/SKU별 판매량 순위, 국가별 배송 비중, 기간 거래(주문→배송) 요약 리포트. "채널별 주문 얼마나 돼", "이번 주 많이 팔린 상품", "SKU별 판매량 top 10", "이번 달 거래 요약해줘", order summary, sales by channel, top selling SKU 요청에 사용.
---

# SmartShip 주문 요약·분석

주문·배송 데이터를 셀러 관점으로 요약한다. 집계 도구가 있는 축은 즉답하고,
상품/SKU 축은 주문 행을 모아 대화 안에서 집계한다(아래 상한 준수).

## 기본 조건

- 기간을 말하지 않으면 **최근 30일 · 등록일(REG_DT) 기준**(주문 관리 화면 기본값)으로 집계하고,
  답변에 어떤 기간·기준을 썼는지 명시한다.
- 모든 요약에는 분모(전체 건수)를 함께 보여준다.

## 1. 채널·상태·국가 축 (집계 도구 즉답)

- 판매채널별: `get_order_counts_by_site`(채널별 주문 건수), `list_order_sites`(채널 목록),
  `get_recent_order_dashboard`(최근 등록 주문의 채널별 집계)
- 주문 상태 funnel: `get_order_count_stats`(SS/FF/SW 상태별) → 등록→확인→출고 단계 요약
- 배송 상태: `get_shipment_status_summary`(배송중/완료/실패/반송), `get_shipment_daily_counts`(일별 추이),
  `get_failed_delivery_counts`(실패 유형별)
- 국가별: `search_shipments`·`search_orders` 행의 출발/도착 국가 필드를 집계하거나 국가 필터로 조회

## 2. 상품/SKU 축 (행 수집 → 대화 내 집계)

서버에 상품 단위 집계 도구가 없으므로 다음 순서로 만든다.

1. `search_orders`로 기간 내 주문을 페이지당 500건씩 수집한다(응답의 total 확인).
2. 수집한 QSP 목록으로 `get_order_goods_by_shipping_nos`를 배치 호출해 상품 행을 받는다.
3. SKU/상품명 기준으로 수량·주문 수를 집계해 상위 N을 표로 보여준다. 채널·국가 분해는
   1의 주문 행과 QSP로 조인한다.

**상한(반드시 지킨다)**: 기간 최대 31일, 주문 **2,000건(500×4페이지)까지**만 수집한다.
초과하면 집계를 중단하지 말고 ① 수집한 범위로 **부분 집계임을 명시**해 답하고
② 기간을 좁히거나 채널을 지정하면 정확해진다고 안내한다. 전수 집계가 필요하면
화면의 엑셀 다운로드를 안내한다.

## 3. 거래 요약 리포트 ("이번 주/이번 달 요약해줘")

같은 기간으로 다음을 병렬 조회해 한 장으로 정리한다:
① 주문 등록·확인 현황(`get_order_count_stats`) ② 채널별 비중(`get_order_counts_by_site`)
③ 배송 상태(`get_shipment_status_summary`) ④ 실패·반송(`get_failed_delivery_counts`)
⑤ (요청 시) 상품 top N(§2) ⑥ TxMoney 차감액(`get_txmoney_monthly_deduction`).

## 규칙

- 집계와 목록은 **같은 기간·같은 기준**으로 조회한다 — 숫자가 화면과 다르면 기간/기준부터 대조.
- 수량 합계인지 주문 건수인지 항상 구분해서 표기한다(한 주문에 여러 상품이 있다).
- 매출 금액 집계는 통화가 섞이므로 통화별로 나눠 보여주고 임의 환산하지 않는다
  (환율이 필요하면 `get_exchange_rates`의 TX 확정 환율을 쓰고 출처를 밝힌다).
- 데이터 변경 도구는 이 스킬 범위 밖이다 — 분석 결과에서 이어지는 처리는 해당 업무 스킬로 넘긴다.
