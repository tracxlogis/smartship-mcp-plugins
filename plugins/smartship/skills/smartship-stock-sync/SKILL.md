---
name: smartship-stock-sync
description: SmartShip 재고 동기화 — SKU↔판매채널 재고 동기화 설정(조회/등록/수정/일괄/삭제/on-off), 수동 동기화 요청, 동기화 이벤트 로그와 SKU별 최종 동기화 내역으로 "채널 재고가 안 맞아요" 진단. stock sync, sync settings, sync history, manual stock sync, "채널에 재고가 잘못 올라가요" 요청에 사용.
---

# SmartShip 재고 동기화

SKU와 판매채널 상품의 재고 동기화 설정·이력 업무를 대화로 수행한다. 순서:
**① 설정 확인 → ② 이벤트 로그·SKU별 내역으로 진단 → ③ (요청 시) 설정 변경·수동 동기화**.

## 화면 기본 조회 조건 (사용자가 조건을 말하지 않으면 이 값을 쓴다)

- **설정 목록**(`get_sync_settings`): 판매채널 전체 · 동기화 유형 전체 · 검색어 타입 `SKU` · 페이지 1,000건
- **동기화 내역**(`get_stock_sync_log`): **오늘 하루 단위**(날짜 1일 조회) · 상태 전체 · 채널 전체.
  다른 날짜를 보려면 날짜를 지정한다

## 1. 설정 확인

- `get_sync_settings` — SKU↔채널 상품 동기화 매핑 목록. 동기화 유형:
  N=동기화 안함 / S=사이트 셋팅 / P=비율 / B=안전재고 / Z=빠른 품절 / L=최소 수량 유지
- 후보값: `get_sync_channel_options`(채널 옵션), `get_sync_channel_sites`(채널 사이트),
  `get_manual_stock_sync_shops`(수동 동기화 가능 샵)

## 2. 진단 ("채널 재고가 안 맞아요")

1. `get_sync_settings` — 해당 SKU의 매핑이 있는지, 동기화 유형이 N(안함)은 아닌지 확인
2. `get_stock_sync_log` — 해당 일자의 동기화 이벤트(성공/실패·전송 수량·결과 메시지) 확인
3. `list_sku_sync_history` — SKU 기준 채널상품별 **최종 동기화 수량·시간** 확인
4. `list_stock_channel_sync_history` — 채널 동기화 이벤트 로그 교차 확인
5. 원인을 좁혀 보고: 매핑 없음 / 동기화 꺼짐 / 전송 실패(메시지) / 채널측 반영 지연

## 3. 설정 변경·수동 동기화 (쓰기 — 전부 confirm=true, 반드시 아래 절차)

변경 내용을 요약해 사용자 동의를 받은 뒤 실행하고, 실행 후 `get_sync_settings`로 재조회해 확인한다.

- `create_sync_setting` / `update_sync_setting` — 매핑 등록/수정
- `set_sync_enabled` — 동기화 on/off 전환
- `bulk_edit_sync_settings` — 일괄 수정(대상 건수를 먼저 보여주고 범위 확정)
- `delete_sync_settings` — 매핑 삭제(삭제하면 그 채널 상품의 재고 동기화가 끊긴다는 영향을 먼저 설명)
- `request_manual_stock_sync` — 수동 동기화 즉시 요청. 반영 결과는 `get_stock_sync_log`로 확인

## 규칙

- **`search_shopee_global_stock`은 조회처럼 보이지만 조건에 따라 매핑을 자동 등록하는 처리 도구다** —
  사용자가 Shopee 글로벌 재고 검색을 명시적으로 요청하고 자동 등록 부수효과에 동의했을 때만 쓴다.
- 동기화는 비동기다 — 수동 동기화 요청 직후 채널에 바로 반영되지 않을 수 있으니
  "요청됨"과 "채널 반영 확인됨"을 구분해 보고한다.
- 현재고 자체가 맞는지는 smartship-sku-stock 스킬(`get_stock_status`)로 먼저 확인한다 —
  원본 재고가 틀리면 동기화 설정을 고쳐도 해결되지 않는다.
