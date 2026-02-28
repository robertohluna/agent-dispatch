# Agent A — BACKEND: Backend Logic

> Sprint 01 | Payment Bug Fix + Checkout Hardening
> Stack: Go + Chi
> Branch: `sprint-01/backend`

## Mission

Fix the webhook handler timeout and refund status desync. Both are handler/service layer issues that depend on DATA's store fixes being merged first.

## Chains

### Chain 1: Webhook Handler Timeout (P1)
**Vector:** `POST /webhooks/stripe` → `webhookHandler.ProcessEvent()` → `paymentService.HandleInvoicePaid()` → `orderStore.MarkPaid()`

**Signal:** 504 after 30s. DATA is fixing the store-layer mutex. Your job: fix the handler-layer timeout and add proper response handling.

**Fix:**
1. `internal/handler/webhook.go` — Add 10s context timeout to `ProcessEvent()`
2. Return 200 to Stripe immediately, process asynchronously if long-running
3. Add structured logging for webhook processing time

**Verify:** `stripe trigger invoice.paid` returns 200 in <2s. Logs show processing time.

### Chain 2: Refund Status Desync (P1)
**Vector:** `POST /api/orders/:id/refund` → `orderHandler.Refund()` → `paymentService.ProcessRefund()` → `stripe.Refunds.Create()` → `orderStore.UpdateStatus()`

**Signal:** Stripe refund succeeds but `UpdateStatus()` fails. Order stuck in "refunding" forever.

**Fix:**
1. `internal/service/payment.go` — Wrap refund + status update in compensating transaction
2. If `UpdateStatus()` fails after Stripe refund succeeds, log critical error + set status to "refund-needs-review"
3. Never leave order in intermediate state

**Verify:** Kill DB mid-refund — order lands in "refund-needs-review", not "refunding". Stripe refund still valid.

## Territory

**Can modify:**
- `internal/handler/webhook.go`
- `internal/handler/order.go`
- `internal/service/payment.go`
- `internal/middleware/`

**Do not touch:**
- `internal/store/` (DATA territory)
- `internal/model/` (DATA territory)
- Stripe client wrapper (SERVICES territory)
- Frontend (FRONTEND territory)
- Test files (QA territory)

## Acceptance Criteria

- [ ] Webhook returns 200 in <2s
- [ ] Refund never leaves order in "refunding" state
- [ ] All error paths produce structured log entries
- [ ] `go build ./...` passes
- [ ] `go test ./internal/handler/... ./internal/service/...` passes
