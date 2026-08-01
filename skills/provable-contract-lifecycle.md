---
name: Manage an Aurora securities-lending contract lifecycle
description: Query open contracts, return borrowed shares, initiate or cancel recalls, and acknowledge executions on the Aurora ATS.
api: https://developer.provablemarkets.com/api/connectapi
protocol: connect
operations:
  - LoginService/LoginWithApiCredentials
  - ContractService/ListContracts
  - ContractService/GetContract
  - ContractService/ListContractHistory
  - ContractService/ReturnContract
  - ContractService/RecallContract
  - ContractService/CancelRecallContract
  - ExecutionService/ListExecutions
  - ExecutionService/AckExecutions
  - ContractEventsService/ContractMade
generated: '2026-07-20'
method: generated
---

# Manage an Aurora contract lifecycle

Contracts are the open interest created when orders match. This flow covers the
post-trade lifecycle: monitoring, returns, and recalls. Contract management runs
04:00–18:00 ET.

## Steps

1. **Authenticate.** `LoginService/LoginWithApiCredentials` → bearer token.
2. **Find contracts.** `ContractService/ListContracts` with filters, then
   `ContractService/GetContract` for detail and
   `ContractService/ListContractHistory` for the event trail.
3. **Acknowledge executions.** `ExecutionService/ListExecutions` then
   `ExecutionService/AckExecutions` to confirm receipt of fills.
4. **Return shares (borrower).** Call `ContractService/ReturnContract`; the lender
   approves via the awaiting-return operations. Watch
   `ContractEventsService/ContractReturnExecuted`.
5. **Recall (lender).** Call `ContractService/RecallContract`; cancel with
   `ContractService/CancelRecallContract` if needed.
6. **Subscribe to events.** Use `ContractEventsService` (Webhook / RPCHooks /
   WebSocket at `wss://provablemarkets.com/api/1/websocket/events`) for
   `ContractMade`, `ContractReturnExecuted`, `ContractRecallExecuted`, etc.

## Rules

- Returns and recalls are asymmetric (borrower returns, lender recalls) with
  approval steps — follow the pending/accepted/rejected event pairs.
- Event delivery is at-least-once; dedupe on contract id + event.
- Settlement follows the DTCC calendar; the venue is closed on US holidays.
