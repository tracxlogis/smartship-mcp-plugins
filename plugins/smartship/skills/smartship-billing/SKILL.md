---
name: smartship-billing
description: SmartShip 정산·인보이스·TxMoney — 잔액·당월 차감예정액, 입출금 내역, 월별 인보이스 드릴다운(마스터→비용유형→건별), 가상계좌·무통장 입금 확인, 바우처, TX 확정 환율. "TxMoney 잔액 얼마야", "이번 달 인보이스 보여줘", "무통장 입금했는데 확인해줘", invoice, TxMoney balance, billing history, bank deposit 요청에 사용.
---

# SmartShip 정산·인보이스·TxMoney

정산·비용 화면(TxMoney 내역·인보이스)의 업무를 대화로 수행한다. 순서:
**① 잔액·차감예정 → ② 입출금 내역 → ③ 인보이스 드릴다운 → ④ (요청 시) 무통장 입금 알림**.

## 화면 기본 조회 조건 (사용자가 조건을 말하지 않으면 이 값을 쓴다)

- **인보이스**(`list_invoices`): 연도 올해 · 월 전체(ALL) · 결제상태 전체
- **차감예정액**(`get_txmoney_monthly_deduction`): 당월 1일~말일(도구 기본값)
- **입출금 내역**(`search_txmoney_history`): 페이지당 **20건 고정**(서비스 계약), 기간 **최대 92일**.
  전체 건수는 `get_txmoney_history_count`로 별도 확인

## 1. 잔액·차감예정

- `get_txmoney_accounts` — TxMoney 계좌 목록·잔액(통화 포함)
- `get_txmoney_monthly_deduction` — 차감예정액. "이번 달 얼마 나가?"의 시작점
- `list_va_accounts` — 입금 대기 가상계좌

## 2. 입출금 내역

- `search_txmoney_history` — 기간·입출 구분(I/O)·거래 유형·검색키(BN=QSP/SZ=송장/RN=참조/HN=내역번호)로
  조회. 환불 포함 검색은 `include_refund=true`
- `get_txmoney_history_count` — 같은 조건의 전체 건수(페이지 안내용)

## 3. 인보이스·바우처·환율

- `list_invoices` — 월별 인보이스 마스터 → `list_invoice_details` — 비용 유형·건별 상세 드릴다운
- `list_vouchers` — 바우처 사용/잔액 내역
- `get_exchange_rates` — TX 확정 환율(KRW 기준). 통화 환산이 필요하면 이 값을 쓰고 출처를 밝힌다

## 4. 무통장 입금 (쓰기 — 반드시 아래 절차)

- `get_txmoney_bank_deposits` — 무통장 입금 신청 내역. 상태: **E=입금 예정, N=입금 완료 알림(확인 요청), Y=확인 완료**
- `notify_bank_deposit_paid`(confirm=true) — E 상태 건에 "입금 완료"를 알린다(E→N).
  **잔액이 바로 충전되는 것이 아니라 운영 확인을 요청하는 알림**이다 — 이 점을 사용자에게 설명하고,
  실제로 입금을 마친 건인지 확인받은 뒤 실행한다. 이미 N/Y 상태인 건에는 실행하지 않는다.

## 규칙

- 금액은 항상 통화와 함께 표기하고, 통화가 섞인 합산·임의 환산을 하지 않는다.
- "돈이 왜 빠졌지?"는 `search_txmoney_history`의 거래 유형(충전/배송비/환불 등)과 QSP 검색(BN)으로
  건별 원인을 찾아 답한다.
- 충전(결제) 실행은 도구가 없다 — TxMoney 화면의 충전 기능을 안내한다.
- API Key 연결에서는 조회만 가능하고, 무통장 입금 알림은 OAuth 연결에서만 실행된다.
