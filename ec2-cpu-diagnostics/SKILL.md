---
name: ec2-cpu-diagnostics
description: >
  Use this skill to investigate EC2 CPU exhaustion and performance
  degradation. Activate when: CPU utilization is high or at 100%,
  instance is slow or unresponsive, CloudWatch CPU alarm has triggered,
  T-series CPU credits are depleted, or the user reports performance
  issues on an EC2 instance. This skill provides structured diagnostic
  procedures including T-series credit analysis, process identification,
  and right-sizing recommendations.
---

# EC2 CPU Diagnostics

## When to use

Any EC2 instance investigation involving high CPU utilization, performance degradation, or CPU credit exhaustion on burstable (T-series) instances.

## Investigation workflow

### Step 1 — Triage

```
# Check CPU utilization trend
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=<id> --period 300 --statistics Average --start-time <time> --end-time <time>

# For T-series instances — check credit balance
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUCreditBalance --dimensions Name=InstanceId,Value=<id> --period 300 --statistics Average --start-time <time> --end-time <time>

aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUSurplusCreditBalance --dimensions Name=InstanceId,Value=<id> --period 300 --statistics Average --start-time <time> --end-time <time>
```

### Step 2 — Identify root cause

- If T-series and CPUCreditBalance is 0: CPU credit exhaustion (see references/D1-cpu-exhaustion.md)
- If SSM available: run `top -bn1 | head -20` to identify consuming process
- Check if spike correlates with SSM session activity, deployments, or cron jobs
- Check CloudTrail for StartSession events around the time of the spike

### Step 3 — Recommend remediation

For credit exhaustion:
- Enable T3 Unlimited mode (warning: charges apply for surplus credits)
- Upgrade instance type (t3.small or larger)
- Switch to compute-optimized (c-series) for sustained workloads

For runaway processes:
- Identify and kill the process
- Investigate what triggered it (operator action, cron, deployment)

## Output format

Provide:
1. Timeline of events (when CPU spiked, what correlated)
2. Root cause with supporting metric data
3. Severity assessment (is this impacting production?)
4. Remediation steps ranked by priority
