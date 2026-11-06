# Chat V1 contract proposal

Status: design draft, 2026-11-06. These files are not active contracts and are not
referenced by consumer generation wrappers. Existing APIs remain unchanged.

- [Browser API](browser-api.openapi.json): conversation and message reads, explicit
  request creation/start/cancellation, read-only SSE subscription and own budget.
- [Assistant API](assistant-api.openapi.json): authenticated internal preparation,
  single-dispatch generation, process-local execution evidence and cancellation.
- [Implementation design](../../../tantusz-design/07-reszletes-design/chat-v1/README.md).

Both files are OpenAPI 3.0.3 JSON documents. JSON is supported by the existing
OpenAPI toolchain; no new runtime format or consumer dependency is introduced.
The proposal must be reviewed and integrated into the active service/internal
contracts before generating application code. Internal APIs must never generate
a browser client. Streaming transport requires an explicit adapter; generated
DTOs remain authoritative and are never manually edited.

## Behavioral contract

Browser authentication uses the existing session and session-bound CSRF header.
Foreign resources return 404. Never trust a browser-supplied user identity.
Creation persists a request but does not call the provider. A connected SSE reader
receives `ready`; only the separate CSRF-protected start operation can prepare,
reserve money and claim dispatch. GET operations never initiate paid work.
Start is asynchronous: accepted orchestration can later fail during preparation
or reservation. The resulting state is available through SSE and status reads.

Only one request can occupy a user's execution slot across devices. Generation
has no automatic retry. Duplicate create/start calls cannot produce another
provider invocation. New manual retries use new client UUIDs and reference the
latest failed turn; previous user text is reused rather than duplicated.

All amounts are USD decimal strings with nine fractional digits. Missing usage is
not zero usage. Execution state and accounting state are independent. An unknown
provider outcome blocks further execution until trusted evidence or an audited
administrative reconciliation establishes completion. Accounting uncertainty can
outlive confirmed execution completion and continues to reserve money.

SSE names match the JSON envelope `type`. Heartbeats on the public stream are
comments. Sequence numbers are connection-local and are not replay cursors.
Delta events contain only newly appended text. Final state is persisted before
the terminal event is emitted. No automatic reconnect/replay in V1. A terminal
request may be read as a single terminal snapshot. Snapshot/message reads recover
the persisted partial content after a disconnect.

The internal preparation plan binds an immutable context and a verified output
cost bound. The backend owns pricing and reservations. An assistant restart
invalidates unconsumed plans; missing process-local execution state never proves
that an earlier Google request was not started. Durable dispatch claims are not
automatically replayed. Internal bearer credentials must not reach the browser.

## Error mapping and transport

- 400: invalid input, malformed cursor, context exceeding the allowed limit.
- 401: no authenticated session or invalid internal service bearer.
- 403: invalid CSRF, disabled identity or forbidden operation.
- 404: missing or foreign resource; missing internal process-local evidence.
- 409: conflicting idempotency key, occupied slot/subscriber, invalid retry/state.
- 429: personal/global budget or configured resource limit prevents new work.
- 503: AI disabled, required configuration missing, provider unavailable before
  streaming headers; known accepted requests retain their inspectable status.

Once SSE headers have been sent, HTTP error status cannot change. Send a sanitized
terminal event when possible and persist the result. Never expose provider error
payloads, prompt text, secrets or stack traces. All private responses are
`Cache-Control: no-store`. Public SSE disables intermediary buffering.

Every documented operation lists the common error envelope. The implementation
must narrow individual conditions consistently with the behavioral contract;
listing an error does not authorize bypassing ownership or starting generation.

## Review and release

Validate both OpenAPI files, test code generation into a temporary directory,
and settle the Spring streaming-controller generation strategy before promoting
the proposal. DTO shape validation does not prove SSE framing or runtime behavior;
those require integration tests through the actual proxies.

No commit, push, submodule update or deployment is performed by this proposal.
Future API-spec commits follow the workspace's explicit commit-message approval
process. Consumer references must then point to the same approved spec commit.
