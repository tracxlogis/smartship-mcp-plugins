---
name: smartship-issue-clinic
description: SmartShip 문제 주문 해결 — 에러/보류 주문이 왜 막혔는지 진단하고 유형별 해결 경로(품절 SKU 매핑·주문 분할, 수취인/주소 수정, 배송사 할당 실패 펜딩 해제, 보류 해제, 배송 중단/재개)로 안내·처리. "주문이 왜 안 나가", "에러 주문 처리해줘", "펜딩 풀어줘", "품절 때문에 막힌 주문", stuck order, pending order, error orders 요청에 사용.
---

# SmartShip 문제 주문 해결

막힌 주문을 **진단 → 유형 판정 → 해결 경로 실행** 순서로 처리한다.
해결 도구는 전부 쓰기(`confirm=true`) — 조치 전에 대상과 방법을 요약 보고하고 동의받는다.

## 1. 진단

- 전체 파악: `get_order_count_stats`의 **ERROR/HOLD 그룹** → `search_orders`에
  대시보드 값(`group_type`/`level2_code`/`level3_code`)을 넘겨 해당 주문 목록 조회
- 건별 원인: `get_order_detail` — 펜딩 사유(pending 설명)·보류 여부·상품 매핑 상태를 읽는다
- 송장 쪽에서 막힌 건: `get_waybill_order_counts`의 실패 집계 →
  `search_waybill_unassignable_orders`(사유·pending_type 포함)

## 2. 유형별 해결 경로

| 증상 | 확인 | 조치(confirm=true) |
|---|---|---|
| 품절/SKU 미매핑 | `get_out_of_stock_skus`, `get_warehouse_sku_qty`(재고 확인) | 다른 SKU로 교체 `map_order_item_sku` · 일부만 출고하려면 분할: `get_order_split_targets` → `get_order_split_items` → `split_order`(복구 `restore_split_order`) |
| 수취인/주소 오류 | `get_order_detail`의 수취인 필드 | `update_recipient_info`(수취인·연락처·주소·통관번호) 또는 `update_order` |
| 배송사 할당 실패(CNSAB/ASCF) | `search_waybill_unassignable_orders`로 사유 확인 | 원인 해소 후 `release_waybill_unassignable_pending` → smartship-waybills 절차로 재할당 |
| 보류(HOLD) | 보류 사유 확인 | `set_order_hold`(flag `D`=해제, `I`=보류) |
| 배송 중단 필요/재개 | 주문 상태 확인 | `stop_delivery` / `resume_delivery` |
| 채널 주문 삭제/복구 | 삭제 경위 확인 | `delete_orders`(신중 — 되돌리기 어려움) |

## 3. 처리 후 확인

- 조치한 건은 **같은 조건으로 재조회**해 실제로 에러/보류 집계에서 빠졌는지 확인하고 결과를 보고한다.
- 해소되지 않는 유형(통관 보류, 창고 측 이슈 등 도구가 없는 영역)은 원인만 정리해주고
  해당 화면 또는 1:1 문의를 안내한다 — 임의로 다른 도구로 우회하지 않는다.

## 규칙

- 진단 없이 조치부터 하지 않는다 — 같은 증상이라도 원인(품절 vs 주소 오류 vs 할당 실패)이 다르다.
- 여러 건을 일괄 조치할 때는 건수와 샘플을 보여주고 범위를 확정받는다.
- 원인·조치·결과를 한 줄씩 남겨 사용자가 무엇이 바뀌었는지 알 수 있게 보고한다.
