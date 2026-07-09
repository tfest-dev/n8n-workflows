# Customer Support Digest

This n8n workflow produces a daily or weekly digest of customer support activity for leadership, customer success, product, and engineering stakeholders.

The flow fetches ticket data, normalises it into a common structure, filters spam and low-signal records, prepares a privacy-aware LLM prompt, generates a Markdown digest, prepares the leadership notification payload, and captures an audit snapshot for review or durable storage.

**Success Path Executed**

<img src=examples/images/flow-executed-n8n-success-path.png alt="Successful execution in n8n on success path"/>

**Failure Path Executed**

<img src=examples/images/flow-executed-n8n-failure-path.png alt="Successful execution in n8n on failure path"/>

## Business value

- Gives founders and customer success leaders a concise view of customer issues without requiring them to work inside the helpdesk.
- Surfaces recurring bugs, UX friction, onboarding confusion, billing questions, and feature requests from real support activity.
- Helps product and engineering teams prioritise work using customer pain signals rather than anecdotal escalation alone.
- Creates a repeatable reporting rhythm for leadership updates, incident review, and product planning.
- Adds control points around source freshness, prompt minimisation, LLM failure handling, delivery idempotency, and audit visibility.

## How the flow works

1. `Manual Trigger (dev)` allows ad-hoc local testing.
2. `Daily Schedule (prod)` is configured for 17:00 server time and can be changed to weekdays or weekly cadence.
3. `Set Example Ticket Inputs` provides sample ticket data, run configuration, delivery target metadata, and source-status simulation controls.
4. `Validate Config / Guardrails` parses input JSON, validates run settings, and creates run-level control metadata, including retry policy, idempotency key, and LLM data-handling policy.
5. `Fetch Helpdesk Tickets (placeholder)` represents the helpdesk data-source layer and returns structured source status: `success`, `partial`, `empty`, or `failed`.
6. `Normalise & Filter Tickets` maps tickets into a common structure, removes spam or weak records, redacts common personal data, classifies broad themes, and counts severity/theme totals.
7. `Prepare Source Alert Payloads` creates operations-alert payloads if the helpdesk source is failed, partial, or empty.
8. `Build Digest Prompt` prepares a minimised LLM prompt using ticket IDs, subjects, short redacted excerpts, customer/account names, plan, priority, severity, tags, and themes.
9. `Call Local LLM` sends the prompt to `http://localhost:5001/completions` with `prompt` and `alias` body fields. Timeout and retry settings are included.
10. `Extract Digest` normalises common local/OpenAI-compatible response shapes into `digest_markdown`.
11. `Prepare LLM Failure Alert` and `Build Deterministic Digest Fallback` handle LLM call or extraction failures by preparing an alert and generating a rule-based fallback digest.
12. `Preview / Save Digest` exposes the digest as `preview_markdown` and marks it as ready for leadership notification.
13. `Prepare Leadership Notification (placeholder)` prepares an email/Slack/Teams-ready payload with an idempotency key.
14. `Log Audit Snapshot (demo)` creates an audit record containing source statuses, alerts, ticket summary, dropped tickets, digest output, notification metadata, and LLM data policy.

## Client-specific implementation requirements

| Area | Required client decision |
| --- | --- |
| Helpdesk source | Replace the placeholder source with Zendesk, Freshdesk, Intercom, Help Scout, Jira Service Management, or authenticated HTTP pagination. |
| Filtering rules | Confirm queues, statuses, priorities, tags, plans, customer segments, and spam/low-signal exclusion rules. |
| LLM endpoint | Confirm local or approved hosted model endpoint, request body shape, authentication, timeout, retry policy, and response field mapping. |
| Data handling | Decide which ticket fields can be sent to the model, whether customer names should be included, and whether additional redaction is required. |
| Delivery | Replace notification placeholder with Slack, Teams, email, or another leadership channel using n8n Credentials. |
| Audit storage | Replace or extend the audit node with database, spreadsheet, document store, object storage, SIEM, or support-system logging. |
| Ownership | Decide who receives digest alerts and who acts on high-severity product/engineering signals. |

## Concurrency and error handling

The starter implementation has one helpdesk source branch because support-ticket data usually comes from a single system of record. A client deployment with multiple sources should split them into separate branches, for example helpdesk, customer success notes, incident tracker, and product feedback board. Each branch should return a structured status object using `success`, `partial`, `empty`, or `failed`.

Current controls included in the workflow:

    - Run-level validation before source collection.
    - Retry and timeout metadata for external API calls.
    - Structured source status with `checked_at`, `completed_at`, `source_timestamp`, record count, warnings, and error message where relevant.
    - Operations-alert payloads for failed, partial, or empty sources.
    - Ticket normalisation and spam/low-signal filtering before the LLM step.
    - Basic redaction of email addresses and phone numbers before prompt construction.
    - LLM error output routed to alert preparation and deterministic fallback digest generation.
    - Delivery idempotency key to reduce duplicate Slack/email posts after retries.
    - Audit snapshot containing source state, dropped tickets, digest generation mode, notification payload, and LLM data policy.

Production hardening recommended:

    - Use real HTTP/native connector retry settings for `429`, `408`, network timeouts, and `5xx` responses.
    - Add pagination and back-off for helpdesk APIs with rate limits.
    - Send source and LLM alerts to an operations channel.
    - Use workflow-level error handling for unexpected node failures.
    - Store audit records outside n8n execution history when digest outputs need retention.
    - Keep idempotency keys on delivery and any follow-up task creation to prevent duplicate notifications.

## Optional LLM usage

The workflow uses deterministic normalisation, filtering, theme hints, severity counts, source-status capture, and fallback digest generation. The LLM is used for wording, grouping, and executive readability.

Recommended LLM data rules:

    - Send only the fields needed for summarisation.
    - Avoid raw ticket transcripts unless there is an explicit business need and an approved model environment.
    - Redact personal data before prompt construction.
    - Prefer account-level summaries over detailed individual-user text.
    - Use a local or approved private model when customer confidentiality requires it.
    - Keep deterministic ticket metadata as the source of truth for counts, severity, and operational status.

## Example data

Example files are in `examples/`:

- `examples/sample_input.md` — sample support tickets and run configuration.
- `examples/sample_output.md` — sample digest output, alert shape, and audit fields.

## Demo notes

To test source-status handling, change `simulate_source_status` in `Set Example Ticket Inputs`:

```json
{"helpdesk":"partial"}
```

or:

```json
{"helpdesk":"failed"}
```

A failed source will still produce a controlled output path with operations alerts and, if the LLM path is unavailable, a deterministic fallback digest.
