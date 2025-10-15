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



"===============================================================
" Z_ALERT_CFG  — Monitor configuration for process chains
"---------------------------------------------------------------
" Purpose
"   Stores config per Process Chain: alert window, frequency,
"   and contact list (multiple emails allowed, separated by ';').
"
" Primary Key
"   CLIENT (MANDT), PROCESS_CHAIN_ID
"
" Fields
"   MANDT               CLNT(3)     " Client
"   PROCESS_CHAIN_ID    CHAR(25)    " Process Chain
"   FREQUENCY           CHAR(10)    " e.g. 'DAILY', 'HOURLY', 'WEEKLY'
"   START_TIME          TIMS(6)     " Optional: start time (HHMMSS)
"   AVAILABILITY_TIME   TIMS(6)     " Expected ready time (HHMMSS)
"   EMAIL_CONTACT       CHAR(254)   " One or many emails; use ';' as separator
"   IS_CRITICAL         CHAR(1)     " Boolean-like: 'X' = critical, ' ' = non-critical
"
" Notes
" - EMAIL_CONTACT accepts ';', ',' and line-breaks (program normalizes to ';')
" - IS_CRITICAL can be used to escalate or route differently
"===============================================================

- Ensure `Z_ALERT_CFG` and `Z_ALERT_LOG` exist with the expected fields.
- Configure SAPconnect (SOST) for outbound email.
- Run `ZPC_MONITOR_ALERT` to queue emails; run `ZPC_DELAY_JUSTIFY` to maintain justifications.



"===============================================================
" Z_ALERT_LOG  — Audit/justification log per process-chain run
"---------------------------------------------------------------
" Purpose
"   Stores one audit record per chain/log execution, including
"   timestamps, status, recipient(s) and the justification text.
"
" Primary Key
"   ID_EVENT (timestamp-based unique ID)
"
" Technical Keys (business key for updates)
"   CHAIN_ID + LOG_ID
"
" Fields
"   ID_EVENT            DEC(15)     " UTC short TS (YYYYMMDDhhmmss)
"   CHAIN_ID            CHAR(25)    " Process Chain
"   LOG_ID              CHAR(25)    " Run Log ID
"   RUN_DATE            DATS(8)     " Execution date (YYYYMMDD)
"   STARTTIMESTAMP      DEC(21,7)   " UTC long TS start (YYYYMMDDhhmmss,mmmmuuu)
"   UPDATEDTIMESTAMP    DEC(21,7)   " UTC long TS last update
"   STATUS              CHAR(1)     " R/X/S/J/G/F/A/Y (RSPC status)
"   AVAILABILITY_TIME   TIMS(6)     " Expected time (HHMMSS)
"   RESPONSIBLE         CHAR(254)   " Email (normalized single address per row)
"   COMMENT_FAIL        CHAR(300)   " Free text justification for delay/failure
"
" Notes
" - Application updates COMMENT_FAIL by CHAIN_ID + LOG_ID
" - ID_EVENT is generated from current timestamp when logging an event
"===============================================================
