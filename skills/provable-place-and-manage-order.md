---
name: Place and manage a securities-lending order on Aurora
description: Authenticate to the Aurora ATS, place a borrow/loan order, then modify, query, or cancel it and follow its lifecycle via OMS events.
api: https://developer.provablemarkets.com/api/connectapi
protocol: connect
operations:
  - LoginService/LoginWithApiCredentials
  - InstrumentService/ListInstruments
  - OmsService/CreateOrder
  - OmsService/GetOrder
  - OmsService/ListOrders
  - OmsService/ModifyOrder
  - OmsService/CancelOrder
  - OmsEventsService/OrderCreated
  - OmsEventsService/OrderExecuted
generated: '2026-07-20'
method: generated
---

# Place and manage an Aurora order

The Aurora API is a **Connect** (Buf) API — call operations as `Service/Method`
over gRPC, gRPC-Web, or Connect-HTTP. All calls after login carry the bearer
access token.

## Steps

1. **Authenticate.** Call `LoginService/LoginWithApiCredentials` with your issued
   API credentials. Store the returned access token (TTL ~1 week) and send it as a
   bearer token on every subsequent call.
2. **Resolve the instrument.** Call `InstrumentService/ListInstruments` to find the
   eligible security you intend to borrow or loan.
3. **Place the order.** Call `OmsService/CreateOrder` with the instrument, side
   (borrow/loan), quantity, and rate terms.
4. **Confirm.** Read `OmsService/GetOrder` (or `OmsService/ListOrders`) to confirm
   the order is resting, or subscribe to `OmsEventsService/OrderCreated`.
5. **Amend or cancel.** Use `OmsService/ModifyOrder` to change price/quantity or
   `OmsService/CancelOrder` to pull it. Order entry/matching runs 04:00–15:00 ET.
6. **Follow execution.** Watch `OmsEventsService/OrderExecuted` for fills;
   executions become contracts (see the contract-lifecycle skill).

## Rules

- Idempotency is not documented — do not blindly retry `CreateOrder`; instead
  reconcile with `ListOrders` before re-sending.
- Errors follow the Connect/gRPC error model (code + message), not problem+json.
- Event delivery is at-least-once and chronological — dedupe on the event/order id.
