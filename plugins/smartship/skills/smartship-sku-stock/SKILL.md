---
name: smartship-sku-stock
description: SmartShip SKU·재고 관리 — SKU 검색/상세, 창고별 재고 수량, 실시간 재고 현황·입출고 이력, 재고 마감 리포트(일/주/월), 판매채널 연결·재고 동기화 내역, SKU 등록/수정/삭제/공유. "이 SKU 재고 몇 개야", "재고 현황 보여줘", "SKU 등록해줘", search skus, stock status, warehouse qty, inventory closing report 요청에 사용.
---

# SmartShip SKU·재고 관리

SKU 관리 화면과 재고 현황 화면의 업무를 대화로 수행한다. 순서:
**① 재고/마감 대시보드 → ② SKU 검색 → ③ SKU 상세·창고별 재고 → ④ (요청 시) 등록/수정/공유**.

## 화면 기본 조회 조건 (사용자가 조건을 말하지 않으면 이 값을 쓴다)

- **SKU 검색**(`search_skus`): 검색 항목 기본 `seller_code`(판매자 정의코드) · 재고/칸닷슈/옵션 종류/
  세트/공유 유형 필터 전체(빈값) · 기간 제한 없음(도구가 화면과 같은 전체 기간을 기본 적용) ·
  페이지 1 / 500건
- **재고 현황**(`get_stock_status`): **창고(`pl_cd`)가 필수** — 사용자가 지정하지 않으면
  `get_fulfillment_warehouses` 의 첫 번째 풀필먼트 센터를 쓰되 어느 창고 기준인지 답변에 밝힌다.
  보기 유형 `SKU_NO` · 검색 유형 `sku_no` · 재고수량 조건 `UP`/0(전체) · 페이지 크기 2000
- **마감 리포트**(`get_daily/weekly/monthly_closing_report`): 기준일 기본 — 일일=어제,
  주간=이번 주, 월간=당월. 센터는 재고 현황과 같은 규칙으로 정한다.
- 화면과 숫자가 다르면 창고 선택과 필터(재고 여부·공유 유형)부터 대조한다.

## 1. 현황 파악 (대시보드·리포트)

- `get_stock_status` — 풀필먼트 센터별 실시간 재고 현황. "재고 현황 보여줘"의 시작점.
- `get_inventory_closing_dashboard` / `get_inventory_closing_report` — 재고 마감 대시보드·리포트.
- `get_daily_closing_report` / `get_weekly_closing_report` / `get_monthly_closing_report` — 일/주/월 마감.
- 창고 후보: `get_fulfillment_warehouses`(풀필먼트 센터), `list_customer_warehouses`(셀러 창고/DPC).

## 2. SKU 검색·상세

- `search_skus` — SKU 메인 그리드 조회(기간·검색어·공유/수동/세트/재고/사은품/카테고리 필터·페이지네이션).
- `get_sku_detail` — SKU 1건 통합 상세(기본정보·창고별 재고·판매채널 연결·공유 파트너·도착보장 재고).
- `get_sku_group_detail` / `get_sku_option_list` / `get_sku_option_detail` — 대표상품(그룹)과 옵션 SKU.
- `get_warehouse_sku_qty` — SKU/대표상품의 창고별 재고 수량만 빠르게 볼 때.
- `lookup_product_by_barcode` — 바코드로 상품 정보 조회(등록/수정 자동 채움 후보).

## 3. 이력·연동 조회

- `get_stock_history` / `get_sku_inout_history` — 입출고 이력.
- `get_sku_cell_stock` — 셀 단위 재고.
- `list_sku_sales_channels` — SKU의 판매채널 상품/옵션 연결과 주문 매핑·재고동기화 상태.
- `list_sku_sync_history` / `list_stock_channel_sync_history` — 재고 동기화 내역·채널 이벤트 로그.
- 세트 SKU: `search_set_skus` / `get_set_sku_components`. 도착보장(칸닷슈/EOT): `get_sku_eot_stock` /
  `list_sku_eot_history`.

## 4. 등록·수정·공유 (쓰기 — 반드시 아래 절차, 전부 `confirm=true`)

실행 전 **변경 내용을 요약해 사용자 동의**를 받는다.

- 등록: `check_sku_product_name_duplicate` 로 중복 확인 → `create_sku_group`(대표상품) →
  `create_sku_option`(옵션).
- 수정: `update_sku_group`, `update_sku_options` — **`update_sku_options` 는 덮어쓰기 성격**이라
  반드시 기존 상세를 먼저 조회해 유지할 필드를 포함해서 보낸다. 조회 없이 부분 필드만 보내지 않는다.
- 삭제: `precheck_delete_skus` 로 재고·주문·세트 포함 여부를 먼저 검사 → 차단 사유가 없을 때만
  `delete_sku_options` / `delete_sku_group`.
- 공유: `check_sku_share_partner` 로 파트너 검증 → `share_skus`, 해제는 `unshare_sku`.
- 도착보장: `set_sku_eot_type`(Y/N 일괄), `adjust_sku_eot_stock`(현재 EOT 재고 확인 후 조정 요청).
- 채널 연결 해제: `unlink_sku_sales_channel` — 해제하면 주문 매핑·재고 동기화가 끊기므로 영향을 먼저 설명한다.

## 규칙

- "재고가 안 맞아요"는 ① `get_stock_status`(현재고) ② `get_stock_history`(입출고 이력)
  ③ `list_sku_sync_history`(채널 동기화) 순으로 어디서 어긋났는지 좁혀 보고한다.
- 수량·금액은 창고/통화 단위와 함께 보고한다.
- 상품 통관 정보 변경, 입고 신청, 반출(RTV)은 별도 화면 업무다 — 도구 범위 밖 요청은 해당 화면을 안내한다.
