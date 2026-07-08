# Webhook Trigger

## Trigger an investigation via Webhook

In production, webhooks enable external systems (monitoring tools, ticketing platforms) to automatically trigger investigations. In this module, you'll configure a webhook and trigger it using `curl` from the EC2 instance's Session Manager terminal — no external tools needed.

### Step 1: Generate webhook credentials

This step is done in the **AWS Management Console** — not the DevOps Agent web app. You'll switch back to the web app in Step 3 to verify the investigation.

1. In the **AWS Management Console**, navigate to **AWS DevOps Agent**.
2. Select your Agent Space.
3. Under **Capabilities**, find **Webhooks** and click **Add webhook**.
4. Click through the prompts including the data schema and creating a secure connection — we won't use these for this workshop.
5. On the final page **"Generate URL and secret key"**, select the option to generate them.
6. **Copy and save** both the URL and secret key shown — you won't see them again.

### Step 2: Trigger via curl

1. Go to **EC2 console** → select **AWS-DevOpsAgent-Test-Instance** → **Connect** → **Session Manager** → **Connect**.
2. Set your webhook variables:

```
WEBHOOK_URL="<paste your webhook endpoint URL>"
SECRET="<paste your secret key>"
```

3. Run the following to trigger an investigation:

```
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%S.000Z)
INCIDENT_ID="webhook-test-$(date +%s)"

PAYLOAD=$(cat <<EOF
{
  "eventType": "incident",
  "incidentId": "$INCIDENT_ID",
  "action": "created",
  "priority": "HIGH",
  "title": "High CPU detected on production EC2 instance",
  "description": "CloudWatch alarm AWS-DevOpsAgent-EC2-CPU-Test triggered. CPU utilization exceeded 70% threshold. Immediate investigation required.",
  "service": "EC2-Workshop-Test",
  "timestamp": "$TIMESTAMP",
  "data": {
    "metadata": {
      "region": "$(curl -s http://169.254.169.254/latest/meta-data/placement/region)",
      "environment": "workshop"
    }
  }
}
EOF
)

SIGNATURE=$(echo -n "${TIMESTAMP}:${PAYLOAD}" | openssl dgst -sha256 -hmac "$SECRET" -binary | base64)

curl -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -H "x-amzn-event-timestamp: $TIMESTAMP" \
  -H "x-amzn-event-signature: $SIGNATURE" \
  -d "$PAYLOAD"
```

4. You should receive a `200` response or output message — this means the webhook was received.

### Step 3: Verify the investigation started

1. Return to the DevOps Agent web app.
2. Click **Incidents**.
3. You should see a new investigation triggered by your webhook with the title "High CPU detected on production EC2 instance".

### How this works in production

```
CloudWatch Alarm → SNS Topic → Lambda → Webhook → Investigation starts
         or
PagerDuty/Datadog/Grafana → Webhook → Investigation starts
         or
ServiceNow ticket created → Built-in integration → Investigation starts
```

Webhooks support two authentication methods:

- **HMAC** (what we just used) — for generic webhooks and Dynatrace
- **Bearer token ** — for Splunk, Datadog, New Relic, ServiceNow, Slack integrations

In production, you'd configure your monitoring tools to call this webhook automatically. The agent starts investigating within seconds of receiving the event. Note: the agent may link this to a previous investigation if it determines they are related.
