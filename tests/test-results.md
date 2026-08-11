# Test Results

## Test 1: SNS Topic and Email Delivery — PASS
- Topic created: `arn:aws:sns:eu-west-3:033216807267:CloudWatchAlerts`
- Email subscribed: `pragash_m@hotmail.co.uk`
- Subscription confirmed via email link, status verified as `Confirmed` in console.

## Test 2: HighCPUUtilization Alarm — PASS
Full state history captured via `describe-alarm-history`:

| Time (UTC+2) | Transition | Reason |
|---|---|---|
| 2026-08-11 22:59:25 | INSUFFICIENT_DATA to OK | Initial evaluation, CPU at ~0.38% |
| 2026-08-11 23:16:25 | OK to ALARM | 2 datapoints [99.99%, 89.16%] both greater than 80% threshold |
| 2026-08-11 (later) | ALARM to OK | CPU dropped back to 69.09%, no longer greater than 80% |

Verified via CLI: state was OK, reason: "Threshold Crossed: 1 datapoint [69.09066693304032 (11/08/26 21:21:00)] was not greater than the threshold (80.0)."

Email notification confirmed received for the ALARM transition. Full OK to ALARM to OK cycle confirmed working end-to-end.

## Test 3: HighMemoryUtilization and LowDiskSpace Alarms — PASS (after fixes)

**Issue found:** CWAgent logs showed repeated AccessDenied errors when attempting cloudwatch:PutMetricData. Root cause: the instance's IAM role (ce-lab-logging-role-pk14) only had a Logs policy, no metrics permission.

**Fix:** Attached AWS managed policy CloudWatchAgentServerPolicy to the role, restarted the agent.

**Issue found:** After the IAM fix, HighMemoryUtilization flipped to OK, but LowDiskSpace stayed on INSUFFICIENT_DATA despite list-metrics confirming the metric was publishing.

**Root cause:** The published disk_used_percent metric carries 4 dimensions (InstanceId, path, device, fstype), but the alarm was only scoped on 2 (InstanceId, path) - CloudWatch requires an exact dimension match.

**Fix:** Deleted and recreated the alarm with all 4 dimensions (device=nvme0n1p1, fstype=ext4).

Final confirmed state: HighMemoryUtilization OK, LowDiskSpace OK - Threshold Crossed: 1 datapoint [34.44378382252442] was not greater than the threshold (80.0).

## Test 4: HighErrorRate Alarm — PASS (after fix)

**Issue found:** Initial curl test against /error returned Express 404 pages (Cannot GET /error) - the route didn't exist in the deployed server.js.

**Fix:** Added an /error route that logs at error level via the existing Winston logger, restarted the Node app.

**Issue found:** Lab template referenced log group /aws/application/api, which doesn't exist in this environment.

**Fix:** Used the actual log group from Lab M6.01, /ce-lab/app-logging, for the metric filter.

After the fix, 20 concurrent requests to /error returned valid JSON, and the alarm transitioned to ALARM shortly after. Email notification confirmed received.

Note: by the time of the final verification pass, the alarm had returned to INSUFFICIENT_DATA - expected, since ErrorCount had no new datapoints after the one-time test burst. This does not indicate a failure; the alarm correctly evaluated a real breach when errors were actually occurring.

## Test 5: Tiered CPU Alarms — PASS (after fix)

**Issue found:** CPU-Warning, CPU-Critical, and LowDiskSpace showed "No actions" in the console, meaning the SNS alarm action was missing entirely from those three alarms.

**Fix:** Re-ran put-metric-alarm for all three with --alarm-actions explicitly included, which updates the existing alarm in place.

Verified fix via CLI: all three alarms (CPU-Critical, CPU-Warning, LowDiskSpace) now show Actions pointing to arn:aws:sns:eu-west-3:033216807267:CloudWatchAlerts.

All three alarms now correctly configured to notify on trigger.

## Summary

All 6 alarms were created, verified against real (not simulated) metric data, and correctly wired to the SNS topic. Two alarms (HighCPUUtilization, HighErrorRate) were pushed into a genuine ALARM state through real load/error generation and confirmed to send email notifications. Three real issues were found and fixed during testing: a missing IAM permission, a dimension mismatch, and two configuration gaps (missing app route, missing alarm actions) - all documented above and in the README.
