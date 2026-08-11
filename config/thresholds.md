# Threshold Decisions and Rationale

## HighCPUUtilization — 80%, 2 evaluation periods (10 min)
Baseline idle CPU on this instance measured well under 5% (confirmed via `get-metric-statistics` before testing). 80% leaves comfortable headroom above normal load while still catching sustained pressure before it becomes critical. Two evaluation periods (10 minutes total) avoid triggering on brief spikes from cron jobs, deploys, or short bursts of traffic.

## CPU-Warning — 70%, 2 evaluation periods (10 min)
Lower threshold than the primary CPU alarm, intended as an early heads-up rather than a page. Same 2-period requirement as the primary alarm so it tolerates the same kind of brief spikes — the goal is "worth keeping an eye on," not urgent action.

## CPU-Critical — 90%, 1 evaluation period (5 min)
Higher threshold, fewer periods required. At 90% sustained CPU, waiting a full 10 minutes to confirm is not appropriate — this tier is meant to fire fast because the instance is close to saturation.

## HighMemoryUtilization — 85%, 2 evaluation periods (10 min)
Memory pressure that crosses 85% risks triggering the OOM killer or severe swapping, so it's set higher than CPU's threshold to avoid alerting on normal memory usage patterns (which fluctuate more than CPU under caching/GC behavior in this app). Two periods used for the same false-positive reasoning as CPU.

## LowDiskSpace — 80%, 1 evaluation period (5 min)
Disk usage is a slow-moving metric — it rarely spikes and then recovers within minutes the way CPU or memory can. A single confirmed reading above 80% is already a real, actionable signal, so there's no benefit to waiting for a second period; doing so would only delay response to a problem that needs lead time to fix (expanding a volume, clearing logs, etc.).

## HighErrorRate — 10 errors per 5 minutes, 1 evaluation period
Application errors are immediately actionable — there's no legitimate reason for a burst of errors to be a "temporary spike" the way a CPU blip can be. A threshold of 10 in a single 5-minute window was chosen to filter out the occasional isolated error (e.g. one bad request) while still catching real problems (a broken deploy, a downstream dependency failing) quickly.
