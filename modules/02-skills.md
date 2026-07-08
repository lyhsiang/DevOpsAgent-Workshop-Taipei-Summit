# Skills

## Upload a Skill and re-investigate

Skills are natural language instructions that teach the agent specialized knowledge. They load automatically when relevant.

### What's in a Skill?

A skill is a zip file containing markdown instructions:

```
ec2-cpu-diagnostics/
├── SKILL.md                          # Main instructions (required)
└── references/
    └── D1-cpu-exhaustion.md          # Detailed runbook
```

### The SKILL.md

The agent reads the `name` and `description` to decide when to activate the skill:

```
---
name: ec2-cpu-diagnostics
description: >
  Use this skill to investigate EC2 CPU exhaustion and performance
  degradation. Activate when: CPU utilization is high or at 100%,
  instance is slow or unresponsive, CloudWatch CPU alarm has triggered,
  T-series CPU credits are depleted, or the user reports performance
  issues on an EC2 instance.
---
```

The body tells the agent *what to do*:

- Check `CPUCreditBalance` and `CPUSurplusCreditBalance` metrics
- Identify if it's a T-series credit exhaustion issue
- Check CloudTrail for `StartSession` events correlating with the spike
- Run `top` via SSM to identify the consuming process
- Recommend specific remediation (T3 Unlimited, instance upgrade, compute-optimized)

### The reference file (D1-cpu-exhaustion.md))

Contains detailed diagnostic procedures:

- Phase 1 (Triage): CloudWatch metrics to check, SSM commands to run
- Phase 2 (Remediate): Actions based on root cause
- Common issues: credit exhaustion patterns, noisy neighbor detection
- Output format: structured root cause + evidence + mitigation

### Step 1: Upload the Skill

[Download EC2 CPU Diagnostics Skill](https://static.us-east-1.prod.workshops.aws/public/a3a381dd-3787-402c-baaf-1f30724eca5f/static/ec2-cpu-diagnostics.zip)

1. Click **Knowledge** → **Skills** → **Add skill**.
2. Upload the `ec2-cpu-diagnostics.zip` file.
3. Leave Agent Type as **All tasks**.
4. Click **Upload**.

### Step 2: Re-run the investigation

1. Navigate back to **Incidents**.
2. Click **Start Investigation**.
3. Use the **exact same prompt** as Module 2:

```
A CloudWatch alarm has triggered on an EC2 instance. Investigate what is happening and determine if action is needed.
   
```
4. Name: `workshop-investigation-2`
5. Click **Investigate**.

As the investigation runs, expand the skills being called to see the agent loading and following your EC2 CPU diagnostics skill in real time.

If the alarm returned to OK, re-run the stress test from Session Manager:

```
CORES=$(nproc) && for i in $(seq 1 $CORES); do (yes > /dev/null) & echo $! >> /tmp/cpu_pids; done && (sleep 300 && kill $(cat /tmp/cpu_pids) 2>/dev/null && rm -f /tmp/cpu_pids) &
```

### Step 3: Compare results

| Aspect | Without Skill | With Skill |
| --- | --- | --- |
| Depth | Surface-level summary | Full diagnostic workflow |
| Structure | Conversational | Systematic triage → remediate |
| Recommendations | Generic | Specific and actionable |
| Skill visibility | No skill loaded | Expands to show skill being used |

<img width="1204" height="637" alt="image" src="https://github.com/user-attachments/assets/252d7e1d-b19a-477f-82b9-b9c70b363481" />


### Key takeaway

Same alarm, same prompt — dramatically different results. The skill told the agent *what to look for* and *how to structure its analysis*. In production, you'd create skills for your database playbooks, deployment rollback procedures, and team tribal knowledge.
