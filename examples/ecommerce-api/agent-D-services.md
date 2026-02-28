# Agent D — SERVICES: Stripe Integration

> Sprint 01 | Payment Bug Fix + Checkout Hardening
> Stack: Go + Stripe SDK
> Branch: `sprint-01/services`

## Mission

Harden the Stripe client to prevent double charges via idempotency keys and add retry logic with backoff for transient failures.

## Chains

### Chain 1: Add Stripe Idempotency Keys (P1)
**Vector:** `paymentService.ChargeCard()` → `stripe.Charges.Create()` → no idempotency key → duplicate charge on retry

**Fix:**
1. `internal/stripe/client.go` — Add `IdempotencyKey` to every create/charge call
2. Key format: `{operation}-{order_id}-{timestamp_hash}`
3. Add idempotency to: `Charges.Create`, `Refunds.Create`, `PaymentIntents.Create`

**Verify:** Call `ChargeCard()` twice with same order — second call returns existing charge, not new one.

### Chain 2: Retry with Backoff (P2)
**Vector:** Stripe returns 429/500 → client returns error immediately → checkout fails

**Fix:**
1. `internal/stripe/client.go` — Add retry wrapper: 3 attempts, exponential backoff (1s, 2s, 4s)
2. Retry on: 429, 500, 502, 503, 504. Do NOT retry on: 400, 401, 402, 404.
3. All retries use same idempotency key (safe because of Chain 1)

**Verify:** Mock Stripe returning 429 twice then 200 — client succeeds on third try. No duplicate charges.

## Territory

**Can modify:**
- `internal/stripe/`

**Do not touch:**
- `internal/handler/` (BACKEND territory)
- `internal/store/` (DATA territory)
- Frontend (FRONTEND territory)

## Acceptance Criteria

- [ ] Every Stripe mutation includes idempotency key
- [ ] Retry logic handles transient failures
- [ ] No retry on client errors (4xx except 429)
- [ ] `go build ./...` passes
- [ ] `go test ./internal/stripe/...` passes
