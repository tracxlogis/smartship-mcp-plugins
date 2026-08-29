---
name: smartship-claims-cs
description: SmartShip 클레임·배송 CS — 판매사이트 클레임(취소/반품/교환) 대시보드·목록·상세 조회, 클레임 승인/거절 처리, 교환 송장 등록, 반품 수거 비용 확인, CS 메모, 수취인 정보 수정. "클레임 처리하자", "반품 요청 들어온 거 봐줘", "교환 송장 등록", claim, return request, delivery CS 요청에 사용.
---

# SmartShip 클레임·배송 CS

배송 CS(클레임) 화면의 업무를 대화로 수행한다. 순서:
**① 대시보드 → ② 클레임 목록 → ③ 건별 상세 확인 → ④ 승인/거절 등 처리**.

## 1. 현황 파악

- `get_cs_delivery_dashboard` — 판매사이트별 클레임 집계. "클레임 들어온 거 있어?"의 시작점.

## 2. 목록·상세

- `search_cs_delivery_orders` / `search_cs_delivery_orders_detailed` — 클레임/CS 주문 검색
  (detailed는 클레임 요약·상품 요약 병합)
- `get_cs_delivery_detail` — QSP 1건의 주문/상품/클레임/관련주문/CS메모 통합
- `get_claim_detail` — 판매사이트 클레임 번호 기준 상품 상세
- `list_cs_memos` — CS 메모 이력

## 3. 처리 (쓰기 — 전부 confirm=true, 반드시 아래 절차)

- **클레임 승인/거절** `process_claim`: 처리 전 ① `get_claim_detail`로 클레임 내용 확인
  ② 거절이면 `get_claim_reject_reasons`로 **해당 채널·유형의 거절 사유 코드를 먼저 조회**해
  사용자가 사유를 고르게 한다 ③ 승인/거절은 판매채널에 통보되어 되돌리기 어렵다 —
  대상 클레임·결정·사유를 요약 보고하고 동의받은 뒤 실행.
- **교환/재배송 송장 등록** `set_claim_tracking_info` — 배송사와 송장번호를 등록/수정.
- **반품 수거(RPC)**: 생성 전 `get_return_shipping_cost`로 **수거 배송비와 TxMoney 잔액을 먼저 확인**해
  비용을 안내한다(잔액 부족이면 충전 안내).
- **수취인 정보 수정** `update_recipient_info` — 수취인/연락처/주소/이메일/통관번호.
- **CS 메모** `set_cs_memo` — 처리 경위를 메모로 남겨달라는 요청에 사용.

## 규칙

- 클레임 유형(취소/반품/교환)과 채널을 먼저 확인하고 답한다 — 채널마다 거절 사유 코드가 다르다.
- 승인/거절 처리 후 같은 조건으로 재조회해 대시보드 건수 변화를 확인·보고한다.
- 반품 수거처럼 비용이 발생하는 처리는 금액·통화를 먼저 보여주고 동의받는다.
- 판매채널 화면에서만 가능한 처리(채널 자체 정책)는 그 사실을 알리고 채널 관리 화면을 안내한다.
