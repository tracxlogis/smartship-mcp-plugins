---
name: smartship-daily-briefing
description: SmartShip 오늘의 업무 브리핑 — 신규/보류/에러 주문, 출하지시 대기, 송장 미출력, 픽업 대기, 배송 실패·폐기 임박, 품절 SKU, 클레임, TxMoney 잔액을 한 번에 모아 우선순위로 보고. "오늘 할 일 뭐야", "업무 브리핑 해줘", "아침 리포트", daily briefing, what should I do today 요청에 사용.
---

# SmartShip 오늘의 업무 브리핑

셀러가 아침에 화면 예닐곱 개를 도는 대신, 처리할 일을 한 번에 모아 **우선순위 순으로** 보고한다.

## 수집 (병렬 호출, 각 화면 기본 기간 사용)

| 항목 | 도구 | 기간 |
|---|---|---|
| 주문 현황(신규/보류/에러) | `get_order_count_stats` | 최근 30일 |
| 품절 SKU | `get_out_of_stock_skus` | 최근 30일 |
| 송장(할당 가능/할당 실패) | `get_waybill_order_counts` | 최근 30일 |
| 픽업 대기 | `list_pickup_waiting_orders` | 최근 30일 |
| 배송 실패 유형별 | `get_failed_delivery_counts` | 최근 30일 |
| 폐기 임박(장기보관) | `get_long_keep_due_orders` | — |
| 클레임/배송 CS | `get_cs_delivery_dashboard` | — |
| TxMoney 잔액 | `get_txmoney_accounts` | — |
| 당월 차감예정액 | `get_txmoney_monthly_deduction` | 당월 |
| (풀필먼트 셀러) 입고 진행 | `get_inbound_request_counts` | — |

## 보고 형식

1. **지금 조치 필요(위험)** — 폐기 임박 건, TxMoney 잔액 < 당월 차감예정액, 할당 실패, 배송 실패.
2. **오늘 처리할 일** — 주문확인 대기, 출하지시 대기, 송장 할당/출력 대기, 픽업 대기, 클레임 답변 대기.
   각 항목에 건수와 **다음 액션 한 줄**(예: "할당 실패 12건 → 사유 확인 후 펜딩 해제 가능")을 붙인다.
3. **참고 현황** — 채널별 신규 주문, 잔액.

- 0건 항목은 생략한다. 항목이 전부 0이면 "처리할 일 없음"을 명확히 말한다.
- 각 항목에서 사용자가 "그거 보여줘/처리해줘"라고 하면 해당 업무 스킬
  (smartship-orders / smartship-waybills / smartship-pickups / smartship-claims-cs /
  smartship-fulfillment-ops)의 절차로 이어간다 — 브리핑 스킬 안에서 바로 쓰기 도구를 호출하지 않는다.

## 규칙

- 도구 하나가 실패해도 나머지 항목은 보고하고, 실패한 항목은 "확인 불가"로 표시한다(0으로 속이지 않는다).
- 건수는 화면 기본 기간 기준임을 하단에 한 줄로 밝힌다.
- 잔액·차감예정액은 통화와 함께 표기한다.
