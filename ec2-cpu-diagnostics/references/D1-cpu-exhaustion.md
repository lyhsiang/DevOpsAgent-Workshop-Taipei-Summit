---
title: "D1 — CPU Exhaustion"
description: "Diagnose high CPU utilization causing instance unresponsiveness or degraded performance"
status: active
severity: HIGH
triggers:
  - "CPUUtilization.*100"
  - "load average.*high"
  - "instance.*slow"
  - "unresponsive"
owner: devops-agent
objective: "Identify the CPU-consuming process and restore normal performance"
context: "Sustained 100% CPU causes instance unresponsiveness, SSH timeouts, and application failures. For T-series instances, CPU credits may be exhausted. For fixed-performance instances, the workload exceeds the instance's compute capacity."
---

## Phase 1 — Triage

MUST:
- Check CloudWatch CPUUtilization: `aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=<id> --period 300 --statistics Average --start-time <time> --end-time <time>`
- For T-series: check CPUCreditBalance and CPUSurplusCreditBalance metrics
- If SSM available: `top -bn1 | head -20` to identify the process consuming CPU
- Check instance type vCPU count vs workload requirements

SHOULD:
- Check if CPU spike correlates with a specific event (deployment, cron job, traffic spike)
- Review CloudWatch CPUUtilization trend over the past 24 hours

MAY:
- Enable detailed monitoring (1-minute intervals) for better granularity
- Check steal time (st in top) — high steal indicates noisy neighbor on shared tenancy

## Phase 2 — Remediate

MUST:
- Identify and address the CPU-consuming process
- For T-series credit exhaustion: switch to unlimited mode or upgrade instance type
- For sustained high CPU: vertical scale (larger instance type) or horizontal scale (add instances)

SHOULD:
- Kill runaway processes if they're not critical
- Optimize application code or queries causing high CPU

## Common Issues

- symptoms: "CPUUtilization at 100%, T2 instance, CPUCreditBalance at 0"
  diagnosis: "T2 CPU credits exhausted. Instance throttled to baseline performance."
  resolution: "Enable T2 unlimited or upgrade to T3/larger instance type."

- symptoms: "High CPU with high steal time (st > 10%)"
  diagnosis: "Noisy neighbor on shared tenancy host."
  resolution: "Stop+start to migrate to different host. Consider dedicated tenancy for consistent performance."

## Output Format

```yaml
root_cause: "<process_runaway|credit_exhaustion|undersized|noisy_neighbor> — <detail>"
evidence:
  - type: cloudwatch_metric
    content: "<CPUUtilization data>"
severity: HIGH
mitigation:
  immediate: "Kill runaway process or upgrade instance"
  long_term: "Right-size instance, optimize application, enable auto-scaling"
```
