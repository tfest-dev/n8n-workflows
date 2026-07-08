# Weekly Business Status Report (local LLM)

This n8n workflow generates a structured weekly business status report from operational notes using a local LLM endpoint, creates a preview/audit payload, optionally passes through a human approval gate, and then hands the report to an email node for delivery.

This workflow is presented as a production-shaped template rather than a fully client-wired integration. The final email and alert delivery nodes remain deployment-specific because credentials, recipients, channels, and retention requirements differ by client.

<img src=examples/images/flow-executed-n8n.png alt="Successful execution in n8n"/>

## What the workflow does

The flow collects weekly business notes, validates that the input is usable, builds a tightly structured Markdown prompt, sends that prompt to a local LLM service, extracts the returned report text, creates a preview/save payload, captures an audit snapshot, passes through an approval checkpoint, and then delivers the report by email.

The repository version uses static example inputs and a local HTTP endpoint. In a client implementation, those inputs would usually come from live systems such as CRM, project-management tools, finance/admin trackers, support systems, or a weekly update form.

## Business value

Weekly reporting is usually repetitive but still needs judgement, structure, and consistency. This workflow gives a client a repeatable reporting pipeline:

    - Operational notes are converted into a leadership-ready report format.
    - Reporting structure stays consistent week to week.
    - Sensitive business context can stay inside the client environment by using a local LLM rather than a third-party hosted model.
    - Teams spend less time formatting updates and more time checking the actual business content.
    - Basic governance is made visible through input guardrails, preview output, audit capture, approval placeholder, and failure-alert handling.

## How the flow works

1. `Manual Trigger (dev)` allows ad-hoc local testing.
2. `Weekly Schedule (prod)` is configured for Monday at 08:00 server time.
3. `Set Raw Inputs` creates example weekly notes for highlights, sales, marketing, operations, product/technology, finance/admin, risks, and next-week priorities.
4. `Validate Inputs / Guardrails` blocks the run if required fields are missing, empty, too short, or collectively too thin to produce a useful report.
5. `Build Prompt` converts the validated fields into a structured Markdown-report prompt.
6. `Call Local LLM` sends the prompt to `http://localhost:5001/completions` with `prompt` and `alias` body fields. The node includes retry and timeout settings and routes failures to the alert path.
7. `Extract Report` normalises common LLM response shapes into a single `report` field. Unexpected response shapes route to the alert path.
8. `Preview / Save Report` preserves the generated report as `preview_markdown`, adds `generated_at`, and marks the payload as `ready_for_email`. This gives the workflow a useful stopping/checkpoint point before delivery.
9. `Log Audit Snapshot (demo)` creates an `audit_record` containing the generated report, report date, generated timestamp, delivery status, and source input data.
10. `Optional Human Approval Gate (demo)` is a pass-through approval placeholder. Replace it with an n8n Wait/Form/Slack approval step when reports go to senior leadership or external recipients.
11. `Send email` is the final delivery step and must be configured with client-specific email credentials, recipients, sender, and subject.
12. `Prepare Failure Alert` and `Failure Alert Output (demo)` create an alert payload if the LLM call fails or the LLM response cannot be extracted.

## Production additions included in this template

This repository version now includes demo-friendly versions of the production controls that should exist in a client implementation:

| Production control | Included node / behaviour | Client implementation note |
| --- | --- | --- |
| Retry and timeout handling | `Call Local LLM` includes timeout and retry settings. | Tune timeout/retry values for the local model, hardware, queueing behaviour, and SLA. |
| LLM failure alerting | `Call Local LLM` and `Extract Report` route error outputs to `Prepare Failure Alert`. | Replace `Failure Alert Output (demo)` with Slack, email, PagerDuty, Opsgenie, or incident-management tooling. |
| Input quality guardrails | `Validate Inputs / Guardrails` checks missing, empty, too-short, and collectively weak inputs. | Replace simple length checks with business-specific validation rules if required. |
| Human approval before delivery | `Optional Human Approval Gate (demo)` passes through by default. | Replace with Wait/Form/Slack approval when sending to senior leadership, board packs, clients, or external recipients. |
| Preview / durable save point | `Preview / Save Report` exposes `preview_markdown`. | Extend or replace with Google Drive, Notion, database, object storage, filesystem, or Slack preview. |
| Audit logging | `Log Audit Snapshot (demo)` creates an `audit_record` in workflow output. | Replace or extend with durable audit storage and retention rules. |

## Client-specific implementation requirements

| Area | Required client decision |
| --- | --- |
| LLM endpoint | Confirm the local model gateway URL, request body shape, authentication, timeout, retry strategy, and expected response field. |
| Inputs | Replace static `Set Raw Inputs` strings with CRM, PM, support, finance, spreadsheet, database, form, Slack, or email ingestion data. |
| Guardrails | Decide minimum required fields, quality thresholds, redaction rules, and whether low-confidence reports should stop or enter review. |
| Failure alerting | Decide which channel receives workflow failure alerts and what operational details should be included. |
| Email delivery | Configure SMTP or transactional email credentials, sender, recipients, and subject. |
| Report governance | Decide who reviews generated reports before sending, or whether reports are safe for automatic delivery. |
| Data retention | Decide whether prompts/reports/source data are logged, where they are stored, who can access them, and how long they are retained. |

## Concurrency and error handling

This is a mostly linear reporting workflow because the report needs a clear source-to-output chain. The high-risk points are input quality, LLM availability, response-shape mismatch, and email delivery.

Current handling:

    - Input guardrails stop weak runs before the LLM call.
    - `Call Local LLM` includes timeout/retry configuration.
    - LLM call failures route to a prepared failure-alert payload.
    - Unexpected LLM response shapes route to a prepared failure-alert payload.
    - `Preview / Save Report` creates a clear checkpoint before delivery.
    - `Log Audit Snapshot (demo)` retains the generated report and source data in the workflow output.
    - `Optional Human Approval Gate (demo)` marks where a real approval workflow should be inserted.

Production hardening still required:

    - Configure the failure-alert output as a real Slack/email/incident-management notification.
    - Configure durable audit storage if the report must be retained outside n8n execution history.
    - Configure the email node credentials, sender, recipients, subject, and body mapping.
    - Decide whether email failures should retry, alert, or create a manual follow-up task.

## Example data

Example input and output files live in `examples/`:

- `examples/sample_notes.md`
- `examples/sample_report_output.md`

The current workflow uses equivalent static notes inside the `Set Raw Inputs` node.