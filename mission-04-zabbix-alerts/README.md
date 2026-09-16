# Mission 04 — Zabbix Alert Triggers & Notifications

## Objective

Configure threshold-based triggers in Zabbix for CPU/memory utilization, verify the full alerting chain (trigger → problem → action), and define a notification action ready for email/SMTP integration.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2)
- **Zabbix version:** 7.0 LTS
- **Host:** Enterprise-D-Bridge

## Steps Performed

### 1. Reviewed built-in template triggers

The `Linux by Zabbix agent` template (applied in Mission 01) ships with pre-built triggers for high CPU load, high memory utilization, and low disk space, visible under **Data collection → Hosts → Enterprise-D-Bridge → Triggers**.

### 2. Created a low-threshold demo trigger

To reliably demonstrate the alerting pipeline without waiting for a real resource spike, a test trigger was created with an easily-reachable threshold:

- **Name:** `Test Alert - CPU load above 0.1 on Enterprise-D-Bridge`
- **Severity:** Warning
- **Expression:**
  ```
  last(/Enterprise-D-Bridge/system.cpu.load[all,avg1])>0.1
  ```

### 3. Verified problem detection

**Monitoring → Problems** confirmed the trigger fired and appeared as an active problem within minutes — validating that metric collection (Mission 01) feeds correctly into Zabbix's trigger evaluation engine.

### 4. Configured a trigger action

**Alerts → Actions → Trigger actions**
- **Name:** `Notify on threshold breach`
- **Condition:** Trigger severity ≥ Warning
- **Operation:** Send message to user group `Zabbix administrators`

No SMTP media type is configured yet, so no email is actually delivered — but **Reports → Action log** confirms the operation is triggered correctly end-to-end. Email delivery can be enabled later by adding an SMTP media type under **Alerts → Media types → Email**, without changing this action's logic.

### 5. Cleaned up

The demo trigger was disabled after validation to avoid a permanently-firing false alarm in the dashboard.

## Result

The full Zabbix alerting pipeline — **metric → trigger → problem → action** — is validated and operational. Notification delivery (email/SMS/webhook) is a drop-in configuration away via Zabbix media types.

## Screenshots

See [`/docs/screenshots/mission-04/`](../docs/screenshots/mission-04/) for the Problems view and Action log confirming the alert fired.
