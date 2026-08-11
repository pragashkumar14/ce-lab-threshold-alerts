# Test Plan

## Objective
Verify that each CloudWatch alarm correctly evaluates its metric, transitions state when the threshold is breached, and triggers a real SNS email notification — not just that the alarm configuration looks correct in the console.

## Test 1: SNS Topic and Email Delivery
1. Create SNS topic, subscribe email address.
2. Confirm subscription via the email link.
3. Verify subscription status shows "Confirmed" in the SNS console.

**Pass criteria:** Subscription status is Confirmed, and test messages published to the topic arrive in the inbox.

## Test 2: HighCPUUtilization Alarm
1. Confirm alarm is created with correct metric (CPUUtilization), threshold (80%), and dimensions (scoped to the correct InstanceId).
2. Confirm initial state is OK or INSUFFICIENT_DATA (before any load).
3. Generate sustained CPU load using `stress --cpu 4 --timeout 900` (15 minutes) on the target EC2 instance.
4. Poll alarm state every 10 seconds using `watch` while load runs.
5. Confirm state transitions OK → ALARM once CPU has exceeded 80% for 2 consecutive 5-minute periods.
6. Confirm an email notification arrives for the ALARM transition.
7. Stop the load, wait for the next evaluation, confirm state transitions back to OK.

**Pass criteria:** Alarm reaches ALARM state under real load, an email notification is received, and it recovers to OK after load stops.

## Test 3: HighMemoryUtilization and LowDiskSpace Alarms
1. Enable CloudWatch Agent metrics collection on the instance (`append-config` with `mem_used_percent` and `disk_used_percent`).
2. Confirm the agent's IAM role has permission to call `cloudwatch:PutMetricData` (checked via agent logs; found and fixed a missing permission — see README).
3. Confirm metrics are actually publishing using `list-metrics` and `get-metric-statistics`, independent of alarm state.
4. Confirm alarm dimensions exactly match the published metric's dimension set (found and fixed a mismatch on LowDiskSpace — see README).
5. Confirm both alarms leave INSUFFICIENT_DATA and reach OK once real data is available and under threshold.

**Pass criteria:** Both alarms show a real evaluated state (OK) backed by actual metric data, not a stale INSUFFICIENT_DATA state.

## Test 4: HighErrorRate Alarm
1. Confirm the target log group (`/ce-lab/app-logging`) exists and is actively receiving logs.
2. Create a metric filter matching `{ $.level = "error" }`.
3. Confirm the application actually has a route that produces an error-level log line (found the `/error` route was missing from the deployed app — added it, see README).
4. Generate 20 concurrent requests to the error-producing route.
5. Wait for logs to flow through the agent to the log group, then check the alarm state.
6. Confirm state transitions to ALARM once ErrorCount exceeds 10 in a 5-minute window.
7. Confirm an email notification arrives.

**Pass criteria:** Alarm reaches ALARM state from real application error logs, and a notification is received.

## Test 5: Tiered CPU Alarms (CPU-Warning / CPU-Critical)
1. Confirm both alarms are created with the correct thresholds (70% / 90%) and evaluation periods (2 / 1).
2. Confirm both alarms have the SNS topic correctly attached as an alarm action (found and fixed a case where three alarms — CPU-Warning, CPU-Critical, LowDiskSpace — were missing this entirely, see README).
3. Confirm both show a real evaluated state (OK), not "No actions" / unconfigured.

**Pass criteria:** Both alarms are correctly wired to notify on trigger, confirmed via `describe-alarms` showing the SNS ARN under `AlarmActions`.
