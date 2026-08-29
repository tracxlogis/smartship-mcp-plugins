---
name: smartship-waybills
description: SmartShip 송장 출력·배송사 할당 — 할당 가능/할당 중/완료/PUDO/실패 섹션 집계, 출력 대기 주문 조회, 배송사 할당(송장 발번)·할당 취소·할당 실패 펜딩 해제, 송장 PDF 출력·재출력, 출력 내역 조회. "송장 뽑아줘", "배송사 할당해줘", "할당 실패 건 다시 풀어줘", "출력 내역 보여줘", waybill, assign courier, print waybill, reprint 요청에 사용.
---

# SmartShip 송장 출력·배송사 할당

송장 출력 화면과 송장 출력 내역 화면의 업무를 대화로 수행한다. 순서:
**① 섹션 집계 → ② 섹션별 목록 → ③ 할당/출력 처리 → ④ 출력 내역·재출력**.

## 화면 기본 조회 조건 (사용자가 조건을 말하지 않으면 이 값을 쓴다)

- **송장 출력**(`get_waybill_order_counts`·`search_waybill_orders`·`search_waybill_unassignable_orders`):
  기간 **최근 30일**(한 번에 최대 31일) · 검색 타입 `sch_type` 기본 `SSON`(주문 번호),
  QSP 다건이면 `BN` + `|` 결합 · 사은품 필터 `sch_apply_filter` 꺼짐(빈값) ·
  주문 조회 방식 `sch_detailed_type` 전체(빈값) · 자체배송 제외(`self_ship_yn` 'N' — 도구가 기본 적용) ·
  페이지 1 / 200건
- **출력 내역**(`list_waybill_print_history`): 기간 **최근 15일** · 검색 타입 `BN` ·
  출력 타입 `print_document` 전체(빈값)
- 집계와 목록은 같은 기간으로 조회한다 — 화면과 숫자가 다르면 기간·검색 타입부터 대조한다.

## 1. 현황 파악 (섹션 집계)

- `get_waybill_order_counts` — 배송사 할당 가능(픽업주소 허브별)/할당 중/할당 완료(도착국·배송사별)/
  PUDO/할당 실패 건수. "송장 뽑을 거 있어?" 질문은 여기서 시작한다.
  할당 실패는 두 갈래다: 목록 안의 fail·pending 과, 별도 집계되는 `unassignable`(CNSAB/ASCF).

## 2. 목록 조회

- `search_waybill_orders` — 출력 대기/완료 주문 목록(페이징). 섹션을 좁힐 때:
  할당 가능 허브는 `sch_addr_no`+`sch_start_nation_cd`, 할당 완료는 `sch_courier`(배송사명)+
  `sch_delivery_nation_cd`, 할당 중/실패는 `sch_courier`에 상태 키워드(requested/processing/fail).
- `search_waybill_unassignable_orders` — 할당 실패(CNSAB/ASCF) 전용 목록. 일반 목록에는 안 보이는
  주문들이다. `shipping_nos` 지정 시 그 건만 정본 재조회(기간 무시).

## 3. 배송사 할당 (쓰기 — 비용 발생, 반드시 아래 절차)

1. 대상 확정: 할당 가능 섹션에서 허브(픽업주소+출발국) 단위로 QSP 목록을 모은다.
2. **사용자에게 요약 보고**: 대상 건수·발송 주소(addr_no)·출발국. 인도(IN) 출발은 TX Money 잔액이
   필요하다는 것을 함께 안내한다.
3. 동의 후 `assign_couriers`(confirm=true) — **1회 100건 이하**로 나눠 호출한다.
   `delivery_service_type`: 인도=DDIR, 국내=DDPC, 국제=ODPC.
4. 결과 해석: `resultCode -101` = TX Money 잔액 부족(충전 안내). `orderResults`의 건별 코드가 정본.
   **응답 성공 ≠ 송장 번호 발급 완료** — 실제 발번은 최대 5분까지 비동기로 이어지며 그동안
   "할당 중" 섹션에 남는다. 최종 확인은 목록 재조회로 한다.
- 할당 취소: `cancel_assign_couriers`(confirm=true) — 주문 상태 RG/D1/OC/OP/ER/D0/PI/P0/P1 건만
  취소 가능하므로 먼저 목록으로 상태를 확인한다. 건별 -20(할당 없음)은 이미 취소된 것과 같다.
- 할당 실패 복구: `release_waybill_unassignable_pending`(confirm=true) — CNSAB/ASCF 펜딩을 해제해
  할당 대기로 되돌린다. 결과 행이 건별 성공 판정의 정본이다.
- 발송지 변경: `update_waybill_sender_info`(confirm=true) — 주소록 값으로 발송인 정보 일괄 갱신.

## 4. 송장 출력·재출력

- 신규 출력: `print_waybills`(confirm=true) — 할당 완료 건의 출력 내역(hist_no)을 새로 만들고
  PDF URL을 반환한다. `courier_nm`은 주문 행의 `transc_nm` 값을 쓴다.
- 재출력: `get_waybill_reprint_url` — 기존 배치(history_seqno)의 PDF URL만 반환(내역 중복 생성 없음).
  **재출력 요청에 print_waybills를 쓰지 않는다.**
- 출력 내역: `list_waybill_print_history`(배치 목록, 행의 seq_no가 history_seqno),
  `list_waybill_print_history_couriers`(배송사 필터 후보), `list_printed_orders`(배치 내 주문).

## 규칙

- 할당은 결제(TX Money 차감)로 이어질 수 있는 **비용 발생 작업**이다 — 동의 없이 실행하지 않고,
  실행 후 성공/실패/잔액부족 건수를 사실대로 보고한다.
- PDF URL에는 사용자 본인의 API Key가 포함된다 — URL을 요약하거나 다른 사람에게 전달하지 않는다.
- 인도(IN) Momoe 라벨(A6/A4)과 패킹 라벨 출력은 아직 도구가 없다 — 송장 출력 화면 이용을 안내한다.
- 주문 보류/해제는 orders 스킬의 `set_order_hold`를 쓴다(flag I=보류, D=해제).
