# Failure Matrix

Load this reference when the proposal includes external calls, async work, queues, retries, or background jobs.

## Checkpoints

- Timeout: What times out, who owns the timeout, and what state remains afterward?
- Retry: Which operations are safe to retry, and what deduplication key is used?
- Duplicate: Can the same request, event, or job be processed twice?
- Ordering: What breaks if messages arrive out of order?
- Partial failure: What happens if step 2 succeeds and step 3 fails?
- Concurrency: What prevents two workers or users from mutating the same record incorrectly?
- Rollback: Is there a true rollback, or only compensation?
- Recovery: How does an operator or scheduled repair bring the system back to a good state?
- Visibility: Which logs, metrics, traces, or audit records expose these failures?

## Red Flags

- Retry without idempotency
- Write-after-side-effect ordering with no compensation
- State transitions that assume exactly-once delivery
- No operator-visible signal for stuck or poison work
- Manual cleanup as the only recovery path
- Rollback language that is not actually implementable in a distributed flow
