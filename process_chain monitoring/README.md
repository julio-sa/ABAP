# BW Process Chain Alerts — ABAP Utilities

This repository contains two ABAP reports that help teams monitor BW process chains and keep audit trails clean and actionable.

## 1) ZPC_MONITOR_ALERT
**Purpose**: Selects today’s process-chain runs and emails the responsible contacts if an issue or a delay is detected.

**How it works**
- Reads configuration from `Z_ALERT_CFG` (availability time, frequency, contacts).
- Reads runtime status from `RSPCLOGCHAIN` / `RSPCPROCESSLOG`.
- Applies simple rules to decide when to notify (error/aborted/skipped/job error; active/ready beyond time; OK/done but late).
- Splits multiple recipients in `email_contact` by `;` (`,` and line breaks supported).
- Sends a single document with one line per recipient in SOST (standard behavior).

**What you’ll see in SOST**
- One line **per recipient** for the same outgoing message (by design).
- Delivery status per address.

**Config you need**
- `Z_ALERT_CFG` with fields like:
  - `PROCESS_CHAIN_ID`, `AVAILABILITY_TIME`, `FREQUENCY`, `EMAIL_CONTACT`, `IS_CRITICAL`
- `Z_ALERT_LOG` to register audit events (chain/log/status/run date/responsible/comment).

## 2) ZPC_DELAY_JUSTIFY
**Purpose**: Quickly update the **Failure Justification** for chain runs already stored in `Z_ALERT_LOG`.

**How it works**
- Selection screen filters (date, chain ID, log ID, responsible, status).
- Classic ALV list with a hotspot on **Failure Justification**.
- Double-click opens a popup to edit, then updates `Z_ALERT_LOG`.

## Security & Privacy
- No hardcoded secrets or hostnames.
- Emails come from your SAPconnect setup; this code only builds the message and recipients.
- To publish publicly, keep real table names/emails out of the code (see “Sanitization” below).

## Sanitization (for public sharing)
- Replace your custom namespaces and table names with neutral ones (e.g., `ZZ2LC_*` → `Z_*`).
- Use `alerts@example.com` for examples.
- Avoid any landscape or company references in comments.

## How to use
- Import the source into your ABAP system (SE38).
- Ensure `Z_ALERT_CFG` and `Z_ALERT_LOG` exist with the expected fields.
- Configure SAPconnect (SOST) for outbound email.
- Run `ZPC_MONITOR_ALERT` to queue emails; run `ZPC_DELAY_JUSTIFY` to maintain justifications.

## License
MIT (or your preferred open-source license)
