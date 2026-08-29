---
name: smartship-orders
description: SmartShip 주문 관리 — 주문 현황 대시보드 집계, 주문 검색/상세 조회, 주문확인·출하지시·보류·삭제·분할 등 주문 처리. "주문 몇 건이야", "펜딩 주문 보여줘", "이 QSP 상세", "주문확인 처리해줘", order status, search orders, order detail, confirm orders, pick and pack 요청에 사용.
---

# SmartShip 주문 관리

셀러의 주문 관리 화면(주문 조회) 업무를 대화로 수행한다. 화면과 같은 순서로 일한다:
**① 대시보드 집계로 현황 파악 → ② 조건을 좁혀 목록 조회 → ③ 건별 상세 → ④ (요청 시) 처리 실행**.

## 화면 기본 조회 조건 (사용자가 조건을 말하지 않으면 이 값을 쓴다)

화면과 같은 기본값으로 조회해야 사용자가 화면에서 보는 숫자와 일치한다.

- 기간 기준 `sch_date_type`: `REG_DT`(등록일) · 기간: **최근 30일** (한 번에 최대 31일)
- 검색 타입 `sch_type`: `BN`(QSP 번호) — 다건은 공백/콤마 입력을 `|` 로 결합
- 페이지: `page_no` 1 / `page_size` 500
- 서비스 타입·등록 방식·국가·창고·운송수단 필터: 전체(빈값)
- 대시보드 집계(`get_order_count_stats`)와 목록(`search_orders`)은 **같은 기간으로** 조회한다 —
  결과 건수가 화면과 다르면 기간·기간 기준부터 대조한다.

## 1. 현황 파악 (대시보드)

- `get_order_count_stats` — SS(일반배송)/FF(풀필먼트)/SW(스마트웨어하우스) 상태별 건수 집계.
  "지금 주문 현황 어때?" 류 질문은 항상 여기서 시작한다.
- `get_order_counts_by_site` / `list_order_sites` — 판매채널(사이트)별 건수와 조회 가능한 채널 목록.
- `get_recent_order_dashboard` — 최근 등록 주문의 채널별 집계.

## 2. 목록 조회

- `search_orders` — 주문 목록 검색. 대시보드 카드 값(`group_type`=FF/SS/ERROR/HOLD,
  `level2_code`, `level3_code`, `order_type`)을 그대로 넘기면 화면과 동일한 조건으로 변환된다.
  대분류 전체는 `scope: "groupTotal"`.
- `get_order_status_search_params` — 대시보드 카드가 실제 어떤 조회 조건(`status`, `order_type_sub`)이
  되는지 조회 없이 확인만 할 때.
- `get_orders_by_shipping_nos` / `get_order_goods_by_shipping_nos` — QSP 번호 목록 기준 주문/상품 행 조회.
- 보조 후보값: `get_out_of_stock_skus`(품절 SKU), `get_delivery_options`, `get_transport_types`.

## 3. 상세 조회

- `get_order_detail` — QSP 1건의 주문 기본정보·상품·배송비/수수료·관련주문 통합 응답.
  사용자가 특정 주문을 지목하면 목록 재조회 대신 이걸 쓴다.

## 4. 주문 처리 (쓰기 — 반드시 아래 절차)

쓰기 도구는 전부 `confirm=true` 가 필요하다. 실행 전에 **대상 QSP 목록과 처리 내용을 요약해 보여주고
사용자의 명시적 동의를 받은 뒤** 호출한다. 동의 없이 먼저 실행하지 않는다.

- `confirm_orders` — 주문확인 처리
- `request_pick_n_pack` / `cancel_pick_n_pack` — 풀필먼트 출하지시 요청/취소
- `set_order_hold` — 보류/보류해제
- `stop_delivery` / `resume_delivery` — 배송중단 요청/배송재개
- `apply_gift_policy` — 사은품 정책 적용
- `delete_orders` — 주문 삭제 (되돌리기 어려움 — 대상을 QSP 단위로 재확인)
- 주문 분할: `get_order_split_targets` → `get_order_split_items` 로 가능 여부 확인 후
  `split_order`, 복구는 `restore_split_order`
- 주문 등록/수정: `create_order`(SS·활성 SW만, FF/BB 불가, `client_request_id` 필요),
  `update_order`(허용된 patch만 병합; items patch·재배송/반송 특수 흐름 미지원),
  `update_order_item`·`map_order_item_sku`(서버 상품 `SEQNO` 로 행 지정)

## 규칙

- 계정 식별(cust_no 등)은 서버가 인증 컨텍스트로 주입한다 — 사용자에게 묻지도, 입력하지도 않는다.
- 대량 처리(수십 건 이상)는 먼저 건수와 샘플 3~5건을 보여주고 범위를 확정받는다.
- 처리 후에는 같은 조건으로 재조회해 결과(성공/실패 건수)를 사실대로 보고한다.
- 도구가 지원하지 않는 요청(예: 반품 등록, 송장 출력)은 해당 화면 이용을 안내하고 임의로 다른 도구로 대체하지 않는다.
