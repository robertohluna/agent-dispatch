# Agent E — QA: Testing & Security

> Sprint 01 | Payment Bug Fix + Checkout Hardening
> Stack: Go testing
> Branch: `sprint-01/qa`

## Mission

Establish test coverage for the payment critical path. Write regression tests for all 3 bugs being fixed. Scan Stripe SDK for known vulnerabilities.

## Chains

### Chain 1: Payment Flow Test Suite (P1)
**Fix:**
1. `internal/store/order_test.go` — Test `MarkPaid()` concurrent access (race detection)
2. `internal/store/order_test.go` — Test `CreateOrder()` duplicate cart rejection
3. `internal/handler/webhook_test.go` — Test webhook returns 200 within timeout
4. `internal/service/payment_test.go` — Test refund with DB failure → lands in review state

**Verify:** `go test -race ./...` — all pass, no races detected.

### Chain 2: Security Scan (P2)
**Fix:**
1. Run `govulncheck ./...` — check Stripe SDK + dependencies for CVEs
2. Verify webhook signature validation exists and is correct
3. Check that Stripe secret key is not logged or exposed in error messages

**Verify:** Zero CVEs. Webhook signature validated. No secrets in logs.

## Territory

**Can create/modify:**
- `internal/store/order_test.go`
- `internal/handler/webhook_test.go`
- `internal/service/payment_test.go`
- `internal/stripe/client_test.go`

**Read-only:** All source code (do not modify application code)

## Acceptance Criteria

- [ ] Regression test for each of the 3 bugs
- [ ] Race condition test for concurrent MarkPaid
- [ ] Race condition test for concurrent CreateOrder
- [ ] `go test -race ./...` passes with 80%+ coverage on payment path
- [ ] Security scan clean
