# Class 05 — AI for Observability & Incident Response
> Original Content | No Copyright | Free to Use and Share

---

## 📌 Overview

```
┌──────────────────────────────────────────────────────────────────┐
│              AI-POWERED OBSERVABILITY STACK                      │
│                                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐  │
│  │   METRICS  │  │    LOGS    │  │   TRACES   │  │  EVENTS  │  │
│  │ Prometheus │  │   Loki /   │  │  Jaeger /  │  │ Alertmgr │  │
│  │  Grafana   │  │    ELK     │  │   Tempo    │  │ PagerDuty│  │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └────┬─────┘  │
│        │               │               │               │         │
│        └───────────────┴───────────────┴───────────────┘         │
│                                │                                 │
│                    ╔═══════════▼═══════════╗                     │
│                    ║     AI ANALYSIS LAYER  ║                     │
│                    ║  • Anomaly Detection   ║                     │
│                    ║  • RCA Generation      ║                     │
│                    ║  • Alert Correlation   ║                     │
│                    ║  • Runbook Lookup      ║                     │
│                    ╚═══════════╤═══════════╝                     │
│                                │                                 │
│             ┌──────────────────┼──────────────────┐              │
│             ▼                  ▼                  ▼              │
│         Auto-fix          Slack Alert        Jira Ticket          │
└──────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts

### AI Log Analysis

```python
#!/usr/bin/env python3
"""AI-powered log analyzer with pattern detection"""

import re
from collections import Counter
from datetime import datetime

LOG_ANALYSIS_PROMPT = """You are an expert SRE analyzing application logs.

Logs:
{logs}

Provide:
1. SEVERITY: Overall severity (Critical/High/Medium/Low)
2. SUMMARY: 2-sentence summary
3. ERRORS: List of unique error types with count
4. ROOT_CAUSE: Most likely root cause
5. IMPACT: User-facing impact
6. IMMEDIATE_ACTION: What to do RIGHT NOW
7. INVESTIGATION: Commands to run for deeper analysis

Format as JSON."""


def preprocess_logs(raw_logs: str) -> dict:
    """Extract key info before sending to AI"""
    lines = raw_logs.split('\n')

    error_pattern = re.compile(r'(ERROR|FATAL|CRITICAL|Exception|Traceback)', re.I)
    warn_pattern = re.compile(r'(WARN|WARNING)', re.I)

    errors = [l for l in lines if error_pattern.search(l)]
    warnings = [l for l in lines if warn_pattern.search(l)]

    # Extract error types
    error_types = Counter()
    for line in errors:
        match = re.search(r'([\w.]+Exception|[\w.]+Error)', line)
        if match:
            error_types[match.group(1)] += 1

    return {
        "total_lines": len(lines),
        "error_count": len(errors),
        "warning_count": len(warnings),
        "top_errors": error_types.most_common(5),
        "sample_errors": errors[:20]  # Limit for LLM context
    }


def ai_analyze_logs(log_file: str, llm_func) -> dict:
    """Full log analysis pipeline"""
    with open(log_file) as f:
        raw = f.read()

    # Pre-process to reduce tokens
    pre = preprocess_logs(raw)

    # Build focused prompt
    prompt = LOG_ANALYSIS_PROMPT.format(
        logs='\n'.join(pre['sample_errors'])
    )

    # AI analysis
    response = llm_func(prompt)
    return {"preprocessing": pre, "ai_analysis": response}


# AI Metric Anomaly Detection
class AIAnomalyDetector:
    """Uses AI to explain statistical anomalies"""

    def __init__(self, llm_func):
        self.llm = llm_func
        self.baselines = {}

    def update_baseline(self, metric_name: str, values: list):
        import statistics
        self.baselines[metric_name] = {
            "mean": statistics.mean(values),
            "stdev": statistics.stdev(values) if len(values) > 1 else 0,
            "p95": sorted(values)[int(len(values)*0.95)]
        }

    def detect_and_explain(self, metric_name: str, current_value: float) -> str:
        baseline = self.baselines.get(metric_name)
        if not baseline:
            return "No baseline established"

        deviation = abs(current_value - baseline["mean"])
        z_score = deviation / baseline["stdev"] if baseline["stdev"] > 0 else 0

        if z_score < 2:
            return "Normal"

        prompt = f"""A metric has an anomaly. Explain possible causes and remediation:

Metric: {metric_name}
Current value: {current_value}
Normal mean: {baseline['mean']:.2f}
Normal p95: {baseline['p95']:.2f}
Z-score: {z_score:.1f} (>2 = anomaly, >3 = severe)

List 3 most likely causes and immediate actions. Be concise."""

        return self.llm(prompt)
```

### AI Incident Response Bot

```python
#!/usr/bin/env python3
"""
AI Incident Response Bot
Automates first-response actions for alerts
"""

import json
import subprocess
from datetime import datetime


INCIDENT_SYSTEM_PROMPT = """You are an expert SRE incident response bot.
When given an alert, you:
1. Assess severity (P1/P2/P3)
2. Identify likely root cause
3. Suggest diagnostic commands (exact kubectl/aws/linux commands)
4. Recommend immediate remediation steps
5. Identify who to escalate to

Always respond in JSON format:
{
  "severity": "P1|P2|P3",
  "assessment": "brief description",
  "root_cause_hypothesis": "most likely cause",
  "diagnostic_commands": ["cmd1", "cmd2"],
  "remediation_steps": ["step1", "step2"],
  "escalate_to": "team/person",
  "auto_remediate": true|false
}"""


class IncidentBot:
    def __init__(self, llm_func, slack_webhook=None, dry_run=True):
        self.llm = llm_func
        self.slack_webhook = slack_webhook
        self.dry_run = dry_run
        self.incident_log = []

    def handle_alert(self, alert: dict) -> dict:
        """Main alert handler"""
        timestamp = datetime.utcnow().isoformat()
        print(f"\n[{timestamp}] Alert received: {alert.get('alertname', 'Unknown')}")

        # AI analysis
        prompt = f"Handle this DevOps alert:\n{json.dumps(alert, indent=2)}"
        ai_response = self.llm(prompt)

        try:
            response = json.loads(ai_response)
        except json.JSONDecodeError:
            response = {"assessment": ai_response, "severity": "P2"}

        # Log incident
        incident = {
            "timestamp": timestamp,
            "alert": alert,
            "ai_response": response
        }
        self.incident_log.append(incident)

        # Take actions based on response
        self._act_on_response(response, alert)

        return response

    def _act_on_response(self, response: dict, alert: dict):
        """Execute safe auto-remediation actions"""
        severity = response.get("severity", "P2")

        # Always notify
        self._notify_slack(response, alert)

        # P1: More aggressive auto-response
        if severity == "P1":
            print(f"  🚨 P1 INCIDENT - Escalating immediately")
            self._create_incident_ticket(response, alert)

        # Auto-remediate only safe actions
        if response.get("auto_remediate") and not self.dry_run:
            for cmd in response.get("diagnostic_commands", [])[:3]:
                if self._is_safe_command(cmd):
                    self._run_command(cmd)

    def _is_safe_command(self, cmd: str) -> bool:
        """Only allow read-only commands"""
        safe_prefixes = ["kubectl get", "kubectl describe", "kubectl logs",
                        "aws cloudwatch", "df -h", "free -h", "ps aux",
                        "netstat", "ss -", "curl -s"]
        return any(cmd.strip().startswith(s) for s in safe_prefixes)

    def _run_command(self, cmd: str) -> str:
        print(f"  Running: {cmd}")
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True, timeout=30)
        return result.stdout + result.stderr

    def _notify_slack(self, response: dict, alert: dict):
        message = (
            f"*Alert*: {alert.get('alertname')}\n"
            f"*Severity*: {response.get('severity')}\n"
            f"*Assessment*: {response.get('assessment')}\n"
            f"*Root Cause*: {response.get('root_cause_hypothesis')}"
        )
        print(f"  Slack notification:\n{message}")

    def _create_incident_ticket(self, response: dict, alert: dict):
        print(f"  Creating Jira ticket for P1 incident...")
        # In production: call Jira API here
```

---

## Hands-On Lab

```bash
#!/bin/bash
# Lab: AI Incident Response Simulation

echo "=== AI Incident Response Lab ==="
echo ""
echo "Scenario: Production database CPU is at 95%"
echo ""

# Simulate alert data
ALERT='{
  "alertname": "HighDatabaseCPU",
  "severity": "critical",
  "instance": "db-prod-01:9100",
  "value": "95.2",
  "duration": "8m"
}'

# Send to local LLM
echo "Sending alert to AI for analysis..."
curl -s http://localhost:11434/api/generate \
  -d "$(python3 -c "
import json
alert = '$ALERT'
payload = {
    'model': 'phi3:mini',
    'stream': False,
    'prompt': '''You are an SRE. Handle this alert and respond with:
1. Severity (P1/P2/P3)
2. Root cause hypothesis
3. Top 3 diagnostic kubectl/linux commands to run
4. Immediate remediation step
Alert: ''' + alert
}
print(json.dumps(payload))
")" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"

echo ""
echo "=== Actual diagnostic commands to run ==="
echo "kubectl get pods -n database -o wide"
echo "kubectl top pods -n database"
echo "kubectl exec -it db-pod -- psql -c 'SELECT pid, query, state FROM pg_stat_activity;'"
```

---

## Interview Questions

### 🎯 Basic Level
```
Q1. What are the Three Pillars of Observability?
    Expected: Metrics (numbers over time), Logs (events/text),
    Traces (request paths through services)

Q2. How does AI improve alert quality vs traditional thresholds?
    Expected: AI detects anomalies contextually, correlates
    multiple signals, reduces false positives, adapts to patterns

Q3. What is MTTR and how does AI reduce it?
    Expected: Mean Time to Resolution; AI speeds up log analysis,
    suggests root cause faster, automates first-response actions

Q4. What is alert fatigue and how does AI solve it?
    Expected: Too many alerts causing engineers to ignore them;
    AI deduplicates, correlates, and prioritizes intelligently

Q5. What is a runbook and how can AI generate one?
    Expected: Step-by-step guide for handling specific alerts;
    AI generates from alert name + service context with commands
```

### 🎯 Intermediate Level
```
Q6. Design an AI-powered alert enrichment pipeline.
    Expected: Alert received → AI fetches related metrics/logs →
    enriches with service context → adds RCA hypothesis →
    routes to right team with priority

Q7. How would you use RAG for incident response?
    Expected: Index all runbooks, past incidents, architecture
    docs as vectors → on alert, retrieve relevant docs →
    feed to LLM for context-aware response

Q8. What is anomaly detection and what AI approach works best?
    Expected: Statistical (Z-score, IQR) for baselines,
    ML models (Isolation Forest, LSTM) for patterns,
    LLMs for explaining anomalies in human terms

Q9. How do you prevent AI from taking harmful auto-remediation actions?
    Expected: Allowlist of safe commands (read-only first),
    human approval for write operations, dry-run mode,
    canary approach (one instance first), timeout + circuit breaker

Q10. Explain correlation vs causation in the context of AI incident analysis.
     Expected: AI may correlate events that happened together
     but aren't causally linked; validate hypotheses with
     timeline analysis and controlled changes
```

### 🎯 Advanced Level
```
Q11. Design a complete AIOps incident response platform.
     Expected: Multi-source ingestion (metrics/logs/traces/events),
     AI correlation engine, RCA with confidence scores,
     RAG-based runbook lookup, auto-remediation with approval gates,
     feedback loop to improve AI recommendations

Q12. How would you build a predictive alerting system
     that catches issues before they occur?
     Expected: Collect historical incident data + leading indicators,
     train time-series ML model, use LLM to explain predictions
     in business terms, trigger proactive investigation,
     measure prediction accuracy and retrain

Q13. What are the challenges of using LLMs for real-time
     incident response?
     Expected: Latency (API calls take 1-5s), hallucination risk,
     context limits for large log files, cost at high alert volume,
     model reliability during own outages

Q14. How would you implement a feedback loop to improve
     AI incident response over time?
     Expected: After each incident, collect engineer feedback
     on AI recommendations (was RCA correct?), store labeled
     incidents, fine-tune or use as few-shot examples,
     measure MTTR improvement over time

Q15. Compare rule-based alerting vs ML anomaly detection
     vs LLM analysis for production use.
     Expected: Rules = fast, predictable, no false negatives for
     known issues; ML = catches unknown patterns, needs training data;
     LLM = best for explanation and correlation, slowest, most expensive;
     use all three in layers
```

---
*Original content | Free to use | No copyright restrictions*

---
---
---

# Class 06 — AIOps: AI for IT Operations
> Original Content | No Copyright | Free to Use and Share

---

## 📌 What is AIOps?

```
┌─────────────────────────────────────────────────────────────────┐
│                    AIOps DEFINITION                              │
│                                                                  │
│  AIOps = AI + ML applied to IT Operations to:                   │
│  • Correlate massive amounts of operational data                │
│  • Detect and predict incidents automatically                   │
│  • Automate root cause analysis                                 │
│  • Reduce noise and alert fatigue                               │
│  • Automate routine IT tasks                                    │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │     DATA     │  │  AI/ML ENGINE│  │    ACTIONS   │          │
│  │              │  │              │  │              │          │
│  │ Metrics      │  │ Anomaly Det. │  │ Alert        │          │
│  │ Logs         │──►│ Correlation  │──►│ Auto-fix     │          │
│  │ Traces       │  │ Prediction   │  │ Ticket       │          │
│  │ Events       │  │ NLP Analysis │  │ Runbook      │          │
│  │ CMDB         │  │ Forecasting  │  │ Escalate     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

---

## AIOps Use Cases

### 1. Intelligent Alert Correlation

```python
#!/usr/bin/env python3
"""
AIOps: Alert Correlation Engine
Groups related alerts into a single incident
"""

from dataclasses import dataclass, field
from datetime import datetime, timedelta
from typing import List, Dict
import json


@dataclass
class Alert:
    id: str
    name: str
    severity: str
    service: str
    timestamp: datetime
    labels: Dict = field(default_factory=dict)
    description: str = ""


@dataclass
class Incident:
    id: str
    alerts: List[Alert] = field(default_factory=list)
    root_alert: Alert = None
    ai_summary: str = ""
    severity: str = "P3"
    created_at: datetime = field(default_factory=datetime.utcnow)


class AIAlertCorrelator:
    """Correlates related alerts using AI and rules"""

    def __init__(self, llm_func, time_window_minutes=10):
        self.llm = llm_func
        self.time_window = timedelta(minutes=time_window_minutes)
        self.active_incidents = []
        self.processed_alerts = []

    def ingest_alert(self, alert: Alert) -> Incident:
        """Process incoming alert and correlate with existing incidents"""
        # Find matching incident (within time window + same service)
        related_incident = self._find_related_incident(alert)

        if related_incident:
            related_incident.alerts.append(alert)
            # Update severity if worse
            if self._severity_rank(alert.severity) > self._severity_rank(related_incident.severity):
                related_incident.severity = alert.severity
            self._update_ai_summary(related_incident)
            return related_incident
        else:
            # Create new incident
            incident = Incident(
                id=f"INC-{len(self.active_incidents)+1:04d}",
                root_alert=alert,
                alerts=[alert],
                severity=alert.severity
            )
            incident.ai_summary = self._generate_ai_summary(incident)
            self.active_incidents.append(incident)
            return incident

    def _find_related_incident(self, alert: Alert) -> Incident:
        """Find incidents within time window with matching service/labels"""
        now = datetime.utcnow()
        for incident in self.active_incidents:
            if (now - incident.created_at) > self.time_window:
                continue
            if incident.root_alert.service == alert.service:
                return incident
            # Check label overlap
            shared_labels = set(incident.root_alert.labels.keys()) & set(alert.labels.keys())
            if shared_labels and any(
                incident.root_alert.labels[k] == alert.labels[k]
                for k in shared_labels
            ):
                return incident
        return None

    def _generate_ai_summary(self, incident: Incident) -> str:
        alert_list = "\n".join([
            f"- {a.name} ({a.severity}) on {a.service}"
            for a in incident.alerts
        ])
        prompt = f"""Summarize this incident in 2 sentences for an SRE:

Alerts:
{alert_list}

Provide: Root cause hypothesis and immediate action."""
        return self.llm(prompt)

    def _update_ai_summary(self, incident: Incident):
        incident.ai_summary = self._generate_ai_summary(incident)

    def _severity_rank(self, sev: str) -> int:
        return {"P1": 4, "P2": 3, "P3": 2, "P4": 1}.get(sev.upper(), 1)


# 2. AI-Powered Capacity Forecasting
class AICapacityForecaster:
    """Predicts resource needs and recommends scaling"""

    def __init__(self, llm_func):
        self.llm = llm_func

    def forecast(self, metric_name: str, historical_data: list,
                 forecast_hours: int = 24) -> str:
        import statistics
        avg = statistics.mean(historical_data[-24:]) if historical_data else 0
        trend = (historical_data[-1] - historical_data[0]) / len(historical_data) if len(historical_data) > 1 else 0

        prompt = f"""Analyze resource usage trend and provide capacity forecast:

Metric: {metric_name}
Current value: {historical_data[-1]:.1f}%
24h average: {avg:.1f}%
Trend (per data point): {trend:+.3f}%
Data points: {len(historical_data)}

Forecast for next {forecast_hours} hours:
1. Will it exceed 85% threshold? When?
2. Recommended action (scale now/monitor/nothing)
3. Recommended scaling adjustment if needed
4. Confidence: High/Medium/Low"""

        return self.llm(prompt)


# 3. AI Change Risk Scorer
class AIChangeRiskScorer:
    """Scores the risk of proposed changes using AI"""

    def __init__(self, llm_func):
        self.llm = llm_func

    def score_change(self, change: dict) -> dict:
        prompt = f"""Score the risk of this infrastructure change:

Change: {json.dumps(change, indent=2)}

Provide JSON response:
{{
  "risk_score": 1-10,
  "risk_level": "Low|Medium|High|Critical",
  "risk_factors": ["factor1", "factor2"],
  "mitigation_required": ["action1", "action2"],
  "rollback_plan": "description",
  "recommended_window": "e.g., off-peak hours, weekend",
  "approval_required": true|false
}}"""

        response = self.llm(prompt)
        try:
            return json.loads(response)
        except:
            return {"risk_level": "Medium", "raw_response": response}
```

### 2. AI for CMDB and Service Discovery

```python
#!/usr/bin/env python3
"""
AI-enhanced CMDB: Auto-discovers service dependencies
and generates documentation
"""

class AIServiceMapper:
    """Uses AI to map service dependencies from logs and traces"""

    def __init__(self, llm_func):
        self.llm = llm_func

    def generate_service_map_doc(self, services: list, dependencies: dict) -> str:
        """Generate human-readable service dependency documentation"""
        dep_str = "\n".join([
            f"  {svc} → {', '.join(deps)}"
            for svc, deps in dependencies.items()
        ])

        prompt = f"""Generate service dependency documentation:

Services: {', '.join(services)}
Dependencies:
{dep_str}

Create:
1. Executive summary of the architecture
2. Single points of failure
3. Blast radius if each service fails
4. Recommended resilience improvements

Format: Clear markdown with sections"""

        return self.llm(prompt)

    def generate_runbook_from_topology(self, failing_service: str,
                                        dependencies: dict) -> str:
        """Generate incident runbook based on service topology"""
        affected = [svc for svc, deps in dependencies.items()
                   if failing_service in deps]

        prompt = f"""Generate an incident runbook:
Failing service: {failing_service}
Directly dependent services: {', '.join(affected)}

Include:
1. Impact assessment
2. Diagnostic steps with commands
3. Remediation options
4. Communication template for stakeholders"""

        return self.llm(prompt)
```

---

## Hands-On Lab

```bash
#!/bin/bash
# Lab: AIOps Alert Simulation

echo "=== AIOps Lab: Alert Correlation Simulation ==="
echo ""

# Simulate 5 correlated alerts arriving
ALERTS=(
    "PodCrashLoopBackOff: payment-api-pod in namespace production"
    "HighErrorRate: payment-api error rate 45% in last 5 minutes"
    "DatabaseConnectionsFull: payment-db at 100% connection pool"
    "HighLatency: payment-api p99 latency > 10s"
    "ServiceUnavailable: payment-api health check failing"
)

for alert in "${ALERTS[@]}"; do
    echo "Alert: $alert"
    RESPONSE=$(curl -s http://localhost:11434/api/generate \
      -d "{\"model\":\"phi3:mini\",\"stream\":false,\"prompt\":\"In 1 sentence, is this alert related to: database connection exhaustion? Alert: $alert\"}" \
      | python3 -c "import sys,json; print(json.load(sys.stdin)['response'].strip())")
    echo "  → AI: $RESPONSE"
    echo ""
done

echo "=== AI Correlation Summary ==="
ALL_ALERTS=$(printf '%s\n' "${ALERTS[@]}")
curl -s http://localhost:11434/api/generate \
  -d "$(python3 -c "
import json
alerts = '''${ALL_ALERTS}'''
print(json.dumps({
    'model': 'phi3:mini',
    'stream': False,
    'prompt': f'Correlate these alerts into 1 incident. Give: root cause, affected service, P1/P2/P3:\n{alerts}'
}))")" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
```

---

## Interview Questions

### 🎯 Basic Level
```
Q1. What is AIOps and how does it differ from traditional monitoring?
    Expected: AIOps = AI/ML applied to IT ops data;
    traditional = threshold-based rules;
    AIOps = learns patterns, correlates signals, predicts

Q2. What is alert correlation and why is it important?
    Expected: Grouping related alerts into one incident;
    reduces noise, identifies root cause faster

Q3. Name 3 AIOps use cases in a production environment.
    Expected: Anomaly detection, alert correlation, capacity
    forecasting, change risk scoring, RCA generation

Q4. What data sources does AIOps typically analyze?
    Expected: Metrics, logs, traces, events, CMDB, change records,
    topology data, deployment history

Q5. What is a CMDB and how does AI enhance it?
    Expected: Configuration Management Database = inventory of IT assets;
    AI auto-discovers services, maps dependencies, detects drift
```

### 🎯 Intermediate Level
```
Q6. How would you implement AI-based change risk scoring
    in a CI/CD pipeline?
    Expected: Before deployment, send change diff to AI with
    context (service criticality, time of day, recent incidents),
    get risk score, block high-risk deploys or require approval

Q7. Explain the difference between reactive, proactive,
    and predictive operations in AIOps.
    Expected: Reactive = alert fires then respond;
    Proactive = detect degradation before outage;
    Predictive = forecast failure days ahead using ML

Q8. How does AIOps reduce Mean Time to Resolution (MTTR)?
    Expected: Automated RCA (minutes vs hours), instant runbook
    retrieval, auto-remediation for known issues, fewer alerts
    to triage, faster escalation to right team

Q9. What are the challenges of implementing AIOps?
    Expected: Data quality issues, model training needs,
    false positive tuning, change management (engineers trust?),
    integration complexity, cost

Q10. How would you measure the ROI of an AIOps implementation?
     Expected: MTTR reduction, alert volume reduction,
     number of incidents auto-resolved, engineer hours saved,
     availability improvement (uptime %),
     cost per incident resolved
```

### 🎯 Advanced Level
```
Q11. Design an AIOps platform architecture for a
     large enterprise with 10,000 services.
     Expected: Streaming ingestion (Kafka), real-time ML
     (anomaly detection), graph-based correlation (service topology),
     LLM layer for explanation/runbooks, human approval workflow,
     feedback loop, multi-tenant with cost attribution

Q12. How would you handle model drift in a production AIOps system?
     Expected: Monitor prediction accuracy over time,
     detect distribution shifts in metrics,
     automated retraining pipeline,
     shadow mode (new model vs old),
     gradual rollout with A/B testing

Q13. Explain how knowledge graphs can enhance AI incident response.
     Expected: Graph represents service dependencies,
     when node fails, graph traversal shows impact radius,
     AI uses graph to contextualize recommendations,
     RCA can follow dependency edges to find root

Q14. How do you handle privacy and compliance when AIOps
     analyzes production logs?
     Expected: PII scrubbing before sending to external AI,
     data retention policies, audit logs of AI decisions,
     private model deployment for regulated industries,
     GDPR/HIPAA compliance in data handling

Q15. Compare AIOps approaches: ML models vs LLMs vs hybrid.
     Expected: ML = fast, deterministic, low cost, needs training data;
     LLM = flexible, explains reasoning, no training needed,
     slower and expensive; Hybrid = ML for detection,
     LLM for explanation, best of both worlds
```

---
*Original content | Free to use | No copyright restrictions*

---
---
---

# Class 07 — AI Agents for DevOps (Hands-On)
> Original Content | No Copyright | Free to Use and Share

---

## 📌 What are AI Agents?

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI AGENT vs LLM                               │
│                                                                  │
│  LLM (Passive):                                                  │
│  Question ──► LLM ──► Answer                                    │
│  (Single turn, no memory, no actions)                           │
│                                                                  │
│  AI AGENT (Active):                                              │
│                                                                  │
│  Goal ──► Plan ──► [Tool Call] ──► [Tool Call] ──► Done         │
│              ↑         │               │                         │
│              └─────────┘               │                         │
│              (Observe & iterate)       │                         │
│                                        │                         │
│  The agent:                            │                         │
│  • Breaks goals into steps       ◄─────┘                         │
│  • Uses tools (kubectl, APIs, shell)                            │
│  • Observes results                                             │
│  • Replans if needed                                            │
│  • Completes the goal autonomously                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## ReAct Agent Pattern

```
┌──────────────────────────────────────────────────────────────┐
│                 ReAct LOOP                                    │
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │  REASON  │───►│   ACT    │───►│ OBSERVE  │              │
│  │(Think    │    │(Use tool,│    │(See what │              │
│  │ what to  │    │ run cmd, │    │ happened)│              │
│  │  do)     │    │ call API)│    │          │              │
│  └──────────┘    └──────────┘    └────┬─────┘              │
│       ▲                               │                     │
│       └───────────────────────────────┘                     │
│                  (Loop until goal met)                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Building a DevOps Agent

```python
#!/usr/bin/env python3
"""
DevOps AI Agent using ReAct pattern
Handles real DevOps tasks autonomously
"""

import subprocess
import json
import re
from typing import Callable, Dict, List

# ── Tool Definitions ──────────────────────────────────────────

def run_kubectl(command: str) -> str:
    """Safe kubectl wrapper (read-only commands only)"""
    ALLOWED = ["get", "describe", "logs", "top", "explain", "api-resources"]
    parts = command.strip().split()
    if len(parts) < 2 or parts[1] not in ALLOWED:
        return f"BLOCKED: Only read-only kubectl commands allowed. Got: {parts[1] if len(parts)>1 else 'none'}"
    try:
        result = subprocess.run(
            f"kubectl {command}", shell=True,
            capture_output=True, text=True, timeout=30
        )
        return result.stdout + (f"\nSTDERR: {result.stderr}" if result.stderr else "")
    except Exception as e:
        return f"Error: {e}"


def run_shell(command: str) -> str:
    """Safe read-only shell commands"""
    SAFE = ["df", "free", "ps", "netstat", "ss", "cat /var/log",
            "tail", "head", "grep", "awk", "wc", "ls", "du"]
    if not any(command.strip().startswith(s) for s in SAFE):
        return f"BLOCKED: Not in safe command list"
    try:
        result = subprocess.run(command, shell=True, capture_output=True, text=True, timeout=15)
        return result.stdout[:2000]  # Limit output
    except Exception as e:
        return f"Error: {e}"


def get_pod_logs(pod_name: str, namespace: str = "default", lines: int = 100) -> str:
    return run_kubectl(f"logs {pod_name} -n {namespace} --tail={lines}")


def describe_resource(resource_type: str, name: str, namespace: str = "default") -> str:
    return run_kubectl(f"describe {resource_type} {name} -n {namespace}")


def get_events(namespace: str = "default") -> str:
    return run_kubectl(f"get events -n {namespace} --sort-by=.metadata.creationTimestamp")


TOOLS = {
    "kubectl": {
        "func": run_kubectl,
        "description": "Run kubectl command (read-only: get, describe, logs, top)"
    },
    "shell": {
        "func": run_shell,
        "description": "Run read-only shell command (df, free, ps, tail, grep, etc.)"
    },
    "pod_logs": {
        "func": lambda args: get_pod_logs(**json.loads(args)),
        "description": "Get logs from a pod. Args: {pod_name, namespace, lines}"
    },
    "describe": {
        "func": lambda args: describe_resource(**json.loads(args)),
        "description": "Describe a K8s resource. Args: {resource_type, name, namespace}"
    },
    "events": {
        "func": lambda args: get_events(**json.loads(args)) if args else get_events(),
        "description": "Get Kubernetes events. Args: {namespace}"
    }
}


# ── ReAct Agent ───────────────────────────────────────────────

class DevOpsAgent:
    def __init__(self, llm_func: Callable, max_iterations: int = 10):
        self.llm = llm_func
        self.max_iterations = max_iterations
        self.system_prompt = self._build_system_prompt()

    def _build_system_prompt(self) -> str:
        tool_descriptions = "\n".join([
            f"  {name}: {info['description']}"
            for name, info in TOOLS.items()
        ])
        return f"""You are an expert DevOps AI agent.
You diagnose and investigate infrastructure issues step by step.

AVAILABLE TOOLS:
{tool_descriptions}

RESPONSE FORMAT - You must use EXACTLY this format:
Thought: [your reasoning about what to do next]
Action: [tool_name]
ActionInput: [input to the tool]

When you have enough information to answer:
Thought: [final reasoning]
Final Answer: [your complete answer with findings and recommendations]

Rules:
- Think before each action
- Use diagnostic commands to gather information
- Be methodical: gather data before concluding
- Never guess; always verify with a tool
- Maximum {self.max_iterations} iterations"""

    def run(self, goal: str) -> str:
        print(f"\n{'='*60}")
        print(f"AGENT GOAL: {goal}")
        print(f"{'='*60}\n")

        conversation = [
            {"role": "system", "content": self.system_prompt},
            {"role": "user", "content": f"Goal: {goal}"}
        ]

        for iteration in range(self.max_iterations):
            print(f"--- Iteration {iteration + 1} ---")

            response = self.llm(
                "\n".join([m["content"] for m in conversation])
            )

            print(f"Agent: {response[:500]}")
            conversation.append({"role": "assistant", "content": response})

            # Check for final answer
            if "Final Answer:" in response:
                final = response.split("Final Answer:")[-1].strip()
                print(f"\n{'='*60}")
                print(f"FINAL ANSWER:\n{final}")
                return final

            # Parse and execute tool action
            action, action_input = self._parse_action(response)
            if action and action in TOOLS:
                print(f"Executing tool: {action}({action_input})")
                observation = TOOLS[action]["func"](action_input)
                obs_truncated = observation[:1000]
                print(f"Observation: {obs_truncated[:200]}...")

                conversation.append({
                    "role": "user",
                    "content": f"Observation: {obs_truncated}"
                })
            else:
                print(f"Warning: Could not parse action from: {response[:100]}")
                break

        return "Max iterations reached. Partial investigation complete."

    def _parse_action(self, response: str):
        action_match = re.search(r'Action:\s*(\w+)', response)
        input_match = re.search(r'ActionInput:\s*(.+?)(?=\nThought|\nAction|\Z)', response, re.DOTALL)

        if action_match and input_match:
            return action_match.group(1).strip(), input_match.group(1).strip()
        return None, None


# Example usage
def demo_agent():
    def simple_llm(prompt: str) -> str:
        # Replace with actual LLM call
        import urllib.request, json
        data = json.dumps({
            "model": "phi3:mini",
            "prompt": prompt,
            "stream": False
        }).encode()
        req = urllib.request.Request(
            "http://localhost:11434/api/generate",
            data=data, headers={"Content-Type": "application/json"}
        )
        with urllib.request.urlopen(req, timeout=90) as r:
            return json.loads(r.read())["response"]

    agent = DevOpsAgent(simple_llm)
    result = agent.run("Investigate why pods in the production namespace may be having issues")
    return result


if __name__ == "__main__":
    demo_agent()
```

---

## Multi-Agent System

```python
#!/usr/bin/env python3
"""
Multi-Agent Incident Response System
Multiple specialized agents collaborate to resolve incidents
"""

class SpecializedAgent:
    def __init__(self, name: str, specialty: str, llm_func):
        self.name = name
        self.specialty = specialty
        self.llm = llm_func

    def analyze(self, context: str) -> str:
        prompt = f"""You are the {self.name} agent specializing in {self.specialty}.

Context: {context}

Provide your specialized analysis and recommendations.
Focus only on {self.specialty} aspects."""
        return self.llm(prompt)


class IncidentOrchestrator:
    """Orchestrates multiple specialized agents"""

    def __init__(self, llm_func):
        self.agents = {
            "infrastructure": SpecializedAgent("Infra", "Kubernetes and container orchestration", llm_func),
            "application": SpecializedAgent("App", "application code and performance", llm_func),
            "database": SpecializedAgent("DB", "database performance and queries", llm_func),
            "network": SpecializedAgent("Net", "networking and connectivity", llm_func),
        }
        self.llm = llm_func

    def handle_incident(self, incident: str) -> str:
        print(f"Orchestrating response to: {incident}\n")

        # Step 1: Gather all specialist analyses
        analyses = {}
        for domain, agent in self.agents.items():
            print(f"Consulting {agent.name} agent...")
            analyses[domain] = agent.analyze(incident)

        # Step 2: Synthesize
        synthesis_prompt = f"""You are the incident commander.
Incident: {incident}

Specialist analyses:
{json.dumps({k: v[:300] for k, v in analyses.items()}, indent=2)}

Synthesize into:
1. Most likely root cause (pick the best hypothesis)
2. Immediate action (one specific action)
3. Owner (which team to assign)
4. Timeline to resolution"""

        return self.llm(synthesis_prompt)
```

---

## Hands-On Lab

```bash
#!/bin/bash
# Lab: Single-step agent interaction

echo "=== DevOps Agent Lab ==="
echo ""
echo "Goal: Diagnose why a pod might be in CrashLoopBackOff"
echo ""

# Step 1: Simulate agent thought
echo "Thought: I should check if there are any failing pods first"
echo "Action: kubectl"
echo "ActionInput: get pods -A --field-selector=status.phase!=Running"
echo ""
kubectl get pods -A --field-selector=status.phase!=Running 2>/dev/null || echo "[No failing pods or kubectl not available]"

echo ""
echo "Thought: Let me check recent events for clues"
echo "Action: events"
kubectl get events -A --sort-by=.metadata.creationTimestamp 2>/dev/null | tail -20 || echo "[Events not available]"

echo ""
echo "Sending gathered data to AI for analysis..."
curl -s http://localhost:11434/api/generate \
  -d '{"model":"phi3:mini","stream":false,"prompt":"A Kubernetes pod is in CrashLoopBackOff. What are the top 5 causes and the exact command to check each one? Format as numbered list."}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
```

---

## Interview Questions

### 🎯 Basic Level
```
Q1. What is an AI agent and how is it different from a chatbot?
    Expected: Agent can take actions, use tools, complete multi-step
    goals autonomously; chatbot only responds to questions

Q2. What is the ReAct pattern in AI agents?
    Expected: Reason-Act-Observe loop;
    think → use tool → see result → think again

Q3. What types of tools can a DevOps AI agent use?
    Expected: kubectl, AWS CLI, shell commands, APIs,
    monitoring systems, ticket systems, code repositories

Q4. Why is safety important in AI agents?
    Expected: Agents can take real actions (delete, scale, restart)
    that affect production; mistakes can cause outages or data loss

Q5. What is an orchestrator in a multi-agent system?
    Expected: Agent that coordinates other specialized agents,
    assigns tasks, collects results, synthesizes final answer
```

### 🎯 Intermediate Level
```
Q6. How would you implement safety guardrails for a
    DevOps AI agent?
    Expected: Allowlist of permitted commands, human approval
    for write/delete operations, dry-run mode, rollback capability,
    audit log of all actions, timeout and circuit breaker

Q7. Explain how memory works in AI agents.
    Expected: Short-term = conversation history in context window;
    Long-term = external vector DB storing past interactions;
    Semantic memory = indexed knowledge base of runbooks/docs

Q8. How would you handle agent failures mid-task?
    Expected: Save state at each step, implement checkpointing,
    retry with exponential backoff, fallback to simpler approach,
    alert human if stuck, never leave infrastructure in bad state

Q9. What is the difference between a reactive agent
    and a planning agent?
    Expected: Reactive = responds to events as they happen;
    Planning = creates a plan for complex goal before acting,
    better for multi-step tasks

Q10. How do you test an AI agent before deploying to production?
     Expected: Simulate environments with mocked tools,
     replay historical incidents to test RCA accuracy,
     red team with edge cases, staged deployment (dev→staging→prod)
```

### 🎯 Advanced Level
```
Q11. Design a self-healing Kubernetes cluster using AI agents.
     Expected: Monitoring agent detects issues, investigation agent
     gathers diagnostics, healing agent applies fixes (with limits),
     verification agent confirms resolution, learning agent updates
     runbooks, all with audit trail and human override

Q12. How would you prevent infinite loops in an AI agent?
     Expected: Max iteration counter, detect repeated tool calls
     with same args, track state changes, timeout per action,
     force final answer after N iterations

Q13. Explain the challenges of multi-agent coordination
     in a DevOps context.
     Expected: Conflicting recommendations, shared resource access,
     communication overhead, consistency of state, partial failures,
     determining ground truth when agents disagree

Q14. How would you implement a learning loop for a DevOps agent?
     Expected: Store successful resolutions with action sequences,
     human feedback on recommendations, fine-tune or few-shot
     with successful examples, A/B test new strategies

Q15. Compare LangChain, CrewAI, and custom agent frameworks
     for DevOps use cases.
     Expected: LangChain = mature, many integrations, complex;
     CrewAI = multi-agent collaboration, role-based;
     Custom = full control, simpler, better for specific use cases;
     choose based on complexity and control requirements
```

---
*Original content | Free to use | No copyright restrictions*

---
---
---

# Class 08 — AI Agents for DevOps (Advanced)
> Original Content | No Copyright | Free to Use and Share

---

## Advanced Agent Patterns

```
┌────────────────────────────────────────────────────────────────┐
│           ADVANCED AGENT ARCHITECTURES                          │
│                                                                 │
│  1. SEQUENTIAL CHAIN                                            │
│     Agent1 ──► Agent2 ──► Agent3 ──► Result                   │
│     (Each agent processes output of previous)                  │
│                                                                 │
│  2. PARALLEL AGENTS                                             │
│     ┌── Agent1 ──┐                                             │
│     ├── Agent2 ──┼──► Aggregator ──► Result                   │
│     └── Agent3 ──┘                                             │
│     (All run simultaneously, results combined)                 │
│                                                                 │
│  3. HIERARCHICAL                                                │
│          Orchestrator                                           │
│         /     |      \                                         │
│     SubA1  SubA2   SubA3                                        │
│     /  \                                                        │
│   Tool  Tool                                                    │
│                                                                 │
│  4. SELF-CRITIQUE LOOP                                          │
│     Agent ──► Output ──► Critic ──► Refined Output             │
│       ▲            └──────────────────────┘                    │
│       └──(if not good enough, revise)                          │
└────────────────────────────────────────────────────────────────┘
```

### GitOps Agent

```python
#!/usr/bin/env python3
"""
GitOps AI Agent: Reviews PRs and generates deployment analysis
"""

import subprocess
import json
from typing import Callable


class GitOpsAgent:
    """AI agent for GitOps workflows"""

    def __init__(self, llm_func: Callable):
        self.llm = llm_func

    def review_pr(self, diff: str, context: str = "") -> dict:
        """Review a pull request diff"""
        prompt = f"""You are a DevOps code reviewer. Review this PR diff:

Context: {context}

Diff:
{diff[:3000]}

Provide JSON:
{{
  "approval": "approve|request_changes|comment",
  "risk_level": "low|medium|high|critical",
  "security_issues": ["issue1"],
  "performance_issues": ["issue1"],
  "best_practice_violations": ["issue1"],
  "missing_tests": ["what to test"],
  "deployment_notes": "special deployment considerations",
  "summary": "2-sentence review summary"
}}"""

        response = self.llm(prompt)
        try:
            return json.loads(response)
        except:
            return {"summary": response, "approval": "comment"}

    def generate_deployment_plan(self, changes: list, environment: str) -> str:
        """Generate a deployment plan for code changes"""
        prompt = f"""Generate a deployment plan for {environment} environment:

Changes:
{json.dumps(changes, indent=2)}

Include:
1. Pre-deployment checklist
2. Deployment steps with commands
3. Verification steps
4. Rollback procedure
5. Communication plan"""

        return self.llm(prompt)

    def analyze_terraform_plan(self, tf_plan_output: str) -> dict:
        """Analyze terraform plan for risks"""
        prompt = f"""Analyze this Terraform plan output for risks:

{tf_plan_output[:4000]}

Return JSON:
{{
  "resources_created": count,
  "resources_modified": count,
  "resources_destroyed": count,
  "risk_score": 1-10,
  "high_risk_changes": ["description"],
  "estimated_cost_change": "estimate or unknown",
  "recommendation": "apply|review|abort",
  "notes": "important considerations"
}}"""

        response = self.llm(prompt)
        try:
            return json.loads(response)
        except:
            return {"recommendation": "review", "notes": response}


# Self-healing Agent
class SelfHealingAgent:
    """Agent that detects and heals infrastructure issues"""

    SAFE_REMEDIATION = {
        "pod_crashloop": "kubectl rollout restart deployment/{name} -n {namespace}",
        "service_unavailable": "kubectl rollout restart deployment/{name} -n {namespace}",
        "high_memory": "kubectl delete pod {name} -n {namespace}",  # Let it restart fresh
    }

    def __init__(self, llm_func: Callable, dry_run: bool = True):
        self.llm = llm_func
        self.dry_run = dry_run
        self.actions_taken = []

    def heal(self, symptom: str, context: str) -> str:
        """Attempt to heal an issue"""
        prompt = f"""You are a self-healing system agent.
Symptom: {symptom}
Context: {context}

Determine:
1. Issue type (from: pod_crashloop, service_unavailable, high_memory, other)
2. Safe remediation available: yes/no
3. Specific resource name and namespace
4. If 'other', explain why manual intervention needed

JSON response:
{{
  "issue_type": "type",
  "safe_remediation": true|false,
  "resource_name": "name",
  "namespace": "ns",
  "manual_reason": "if not safe"
}}"""

        analysis = json.loads(self.llm(prompt))

        if not analysis.get("safe_remediation"):
            return f"Manual intervention needed: {analysis.get('manual_reason')}"

        issue_type = analysis["issue_type"]
        if issue_type not in self.SAFE_REMEDIATION:
            return f"No safe remediation for: {issue_type}"

        cmd = self.SAFE_REMEDIATION[issue_type].format(**analysis)

        if self.dry_run:
            result = f"[DRY-RUN] Would execute: {cmd}"
        else:
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
            result = result.stdout or result.stderr

        self.actions_taken.append({"symptom": symptom, "command": cmd, "result": result})
        return result
```

---

## Hands-On Lab

```bash
#!/bin/bash
# Lab: GitOps Agent Simulation

echo "=== GitOps Agent Lab ==="
echo ""
echo "Simulating PR review for a Kubernetes deployment change"

# Sample diff
DIFF='
-        replicas: 1
+        replicas: 10
-          memory: "128Mi"
-          cpu: "100m"
+          memory: "8Gi"
+          cpu: "4"
+        env:
+        - name: DB_PASSWORD
+          value: "hardcoded-secret-123"
'

echo "PR Diff:"
echo "$DIFF"
echo ""
echo "AI Review:"

curl -s http://localhost:11434/api/generate \
  -d "$(python3 -c "
import json
diff = '''$DIFF'''
print(json.dumps({
    'model': 'phi3:mini',
    'stream': False,
    'prompt': f'Review this Kubernetes YAML diff. List: security issues, performance concerns, approval decision:\n{diff}'
}))")" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
```

---

## Interview Questions

### 🎯 All Levels
```
Q1. What is a self-healing agent and what are its safety requirements?
    Expected: Automatically detects and fixes known issues;
    safety: allowlist of remediation actions, dry-run first,
    human escalation for unknown issues, audit trail

Q2. How does a GitOps AI agent improve the deployment process?
    Expected: Auto-reviews PRs for issues, generates deployment plans,
    analyzes Terraform plans for risk, creates rollback procedures

Q3. What is the self-critique pattern and when is it useful?
    Expected: Agent generates output → critic agent reviews it →
    original agent revises; useful for code generation, runbooks,
    complex analysis where quality matters

Q4. How would you implement a parallel multi-agent system?
    Expected: Fan out task to multiple specialized agents,
    gather all responses, aggregator synthesizes best answer,
    compare results for consistency

Q5. What observability do you need for a production AI agent?
    Expected: Log all tool calls and results, track token usage/cost,
    measure task success rate, alert on failures, latency monitoring,
    human review of decisions

Q6. How do you version control AI agent behavior?
    Expected: Version system prompts in git, track tool definitions,
    test with eval suites before promoting, maintain changelog,
    ability to rollback prompt version

Q7. What is the difference between an agent tool and an MCP server?
    Expected: Tool = function defined in code, called by agent;
    MCP = standardized protocol for tool servers, enables
    sharing tools across different AI systems

Q8. How do you handle rate limits when an agent makes
    many API calls?
    Expected: Exponential backoff, request queuing, caching
    repeated calls, fallback to simpler local tools,
    budget monitoring to stop before hitting limits

Q9. Design an agent that handles on-call escalation.
    Expected: Receive alert → gather diagnostics → check runbooks →
    attempt auto-remediation → if failed, create incident ticket →
    page on-call with summary → send Slack notification with context

Q10. What are the ethical considerations for autonomous agents
     in production infrastructure?
     Expected: Audit trail for all actions, human override always possible,
     no autonomous access to financial resources, explain decisions,
     bias in training data affecting decisions, accountability chain
```

---
*Original content | Free to use | No copyright restrictions*

---
---
---

# Class 09 — AI for Security & FinOps (Cost Optimization)
> Original Content | No Copyright | Free to Use and Share

---

## 📌 Part A: AI for DevSecOps

```
┌──────────────────────────────────────────────────────────────────┐
│                  AI-POWERED SECURITY PIPELINE                    │
│                                                                  │
│  CODE          BUILD           DEPLOY          RUNTIME           │
│  ──────        ──────          ──────          ──────            │
│  AI SAST       AI Container    AI Policy       AI Threat         │
│  Code Review   Scanning        Enforcement     Detection         │
│  IaC Scanning  Secret Det.     RBAC Review     Anomaly Det.      │
│                                                                  │
│  Tools:                                                          │
│  Semgrep+AI    Trivy+AI        OPA+AI          Falco+AI          │
│  Snyk AI       Grype           Kyverno+AI      Sysdig AI         │
└──────────────────────────────────────────────────────────────────┘
```

### AI Security Scanner

```python
#!/usr/bin/env python3
"""AI-powered security scanner for DevOps artifacts"""

import json
import re
from typing import Callable


class AISecurityScanner:
    """Scans code, configs, and IaC for security issues"""

    SEVERITY_ORDER = ["CRITICAL", "HIGH", "MEDIUM", "LOW", "INFO"]

    def __init__(self, llm_func: Callable):
        self.llm = llm_func

    def scan_dockerfile(self, content: str) -> list:
        prompt = f"""Security audit this Dockerfile. Find ALL issues.

Dockerfile:
{content}

Check for:
- Running as root (no USER instruction)
- Exposed sensitive ports (22, 23, 3306)
- Secrets in ENV or ARG
- Latest/no version pinning
- COPY . . (copies everything including .git, secrets)
- Curl piped to bash patterns
- Missing HEALTHCHECK
- Unnecessary packages

Return JSON array:
[{{"issue": "description", "severity": "CRITICAL|HIGH|MEDIUM|LOW", "line": N, "fix": "how to fix"}}]"""

        response = self.llm(prompt)
        try:
            start = response.find('[')
            end = response.rfind(']') + 1
            return json.loads(response[start:end])
        except:
            return [{"issue": response, "severity": "MEDIUM", "fix": "Manual review needed"}]

    def scan_kubernetes_yaml(self, yaml_content: str) -> list:
        prompt = f"""Security audit this Kubernetes YAML.

YAML:
{yaml_content[:3000]}

Check:
- Container running as root (runAsUser: 0 or missing runAsNonRoot)
- Privileged containers
- AllowPrivilegeEscalation: true
- Missing resource limits (CPU/memory)
- Secrets in environment variables
- hostNetwork/hostPID/hostIPC: true
- Writable root filesystem
- Capabilities not dropped
- Missing NetworkPolicy

Return JSON array:
[{{"issue": "description", "severity": "CRITICAL|HIGH|MEDIUM|LOW", "fix": "fix"}}]"""

        response = self.llm(prompt)
        try:
            start = response.find('[')
            end = response.rfind(']') + 1
            return json.loads(response[start:end])
        except:
            return []

    def scan_terraform(self, tf_content: str) -> list:
        prompt = f"""Security audit this Terraform code.

Code:
{tf_content[:3000]}

Check:
- S3 buckets with public access
- Security groups with 0.0.0.0/0 on sensitive ports
- Unencrypted storage (EBS, RDS, S3)
- Hardcoded credentials or secrets
- Missing MFA delete on S3
- IAM policies with * actions or resources
- Missing VPC flow logs
- Publicly accessible RDS instances
- Unencrypted data at rest and transit

Return JSON: [{{"issue": "...", "severity": "...", "resource": "...", "fix": "..."}}]"""

        response = self.llm(prompt)
        try:
            start = response.find('[')
            end = response.rfind(']') + 1
            return json.loads(response[start:end])
        except:
            return []

    def detect_secrets(self, content: str, filename: str = "") -> list:
        """Detect potential secrets/credentials in content"""
        # Pre-filter with regex for common patterns
        patterns = {
            "AWS Access Key": r'AKIA[0-9A-Z]{16}',
            "Generic API Key": r'api[_-]?key[_-]?=\s*["\'][a-zA-Z0-9]{20,}["\']',
            "Private Key": r'-----BEGIN (RSA |EC )?PRIVATE KEY-----',
            "Password in code": r'password\s*=\s*["\'][^"\']{8,}["\']',
            "JWT Token": r'eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}\.'
        }

        found = []
        for secret_type, pattern in patterns.items():
            matches = re.findall(pattern, content, re.IGNORECASE)
            if matches:
                found.append({
                    "type": secret_type,
                    "count": len(matches),
                    "severity": "CRITICAL",
                    "file": filename,
                    "fix": "Remove secret, rotate immediately, use secrets manager"
                })

        return found

    def generate_security_report(self, scan_results: dict) -> str:
        total_issues = sum(len(v) for v in scan_results.values())
        critical = sum(
            1 for issues in scan_results.values()
            for issue in issues
            if issue.get("severity") == "CRITICAL"
        )

        prompt = f"""Generate a security report summary:

Total issues: {total_issues}
Critical: {critical}
Scan results by category: {json.dumps({k: len(v) for k, v in scan_results.items()})}

Top issues: {json.dumps([i for issues in scan_results.values() for i in issues[:2]], indent=2)[:1000]}

Write:
1. Executive summary (3 sentences)
2. Risk level: Critical/High/Medium/Low
3. Top 3 priority fixes
4. Compliance implications (SOC2, PCI, HIPAA if relevant)"""

        return self.llm(prompt)
```

---

## 📌 Part B: AI for FinOps

```
┌──────────────────────────────────────────────────────────────────┐
│                  AI-POWERED FINOPS                               │
│                                                                  │
│  VISIBILITY         OPTIMIZATION         GOVERNANCE              │
│  ──────────         ────────────         ──────────              │
│  AI cost            Right-sizing         Budget                  │
│  attribution        recommendations      anomaly det.            │
│                                                                  │
│  Waste              Reserved             Chargeback              │
│  detection          Instance AI          automation              │
│                     advisor                                      │
│                                                                  │
│  Tagging            Spot Instance        Forecast vs             │
│  compliance         optimization         actual alerts           │
└──────────────────────────────────────────────────────────────────┘
```

### AI FinOps Analyzer

```python
#!/usr/bin/env python3
"""AI-powered cloud cost optimizer"""

import json
from typing import Callable


class AIFinOpsAnalyzer:
    """Analyzes cloud costs and recommends optimizations"""

    def __init__(self, llm_func: Callable):
        self.llm = llm_func

    def analyze_aws_cost_report(self, cost_data: dict) -> str:
        prompt = f"""Analyze this AWS cost report as a FinOps expert:

Data: {json.dumps(cost_data, indent=2)[:2000]}

Provide:
1. Top 3 cost drivers
2. Waste identified (idle resources, oversized instances)
3. Immediate savings opportunities ($ and %)
4. Reserved Instance/Savings Plan recommendations
5. Right-sizing recommendations
6. Tagging issues affecting cost attribution
7. 30-day cost forecast

Format with clear sections and dollar amounts where possible."""

        return self.llm(prompt)

    def rightsize_recommendation(self, instance_metrics: dict) -> dict:
        prompt = f"""Recommend right-sizing for this EC2 instance:

Metrics (30-day):
{json.dumps(instance_metrics, indent=2)}

Return JSON:
{{
  "current_type": "t3.xlarge",
  "recommended_type": "t3.medium",
  "reason": "CPU avg 12%, memory avg 35%",
  "monthly_savings": "$45.60",
  "risk": "Low - same family, less CPU",
  "action": "resize|terminate|schedule|keep"
}}"""

        response = self.llm(prompt)
        try:
            start = response.find('{')
            end = response.rfind('}') + 1
            return json.loads(response[start:end])
        except:
            return {"action": "review", "reason": response[:200]}

    def detect_cost_anomalies(self, daily_costs: list, service_name: str) -> str:
        import statistics
        if len(daily_costs) < 7:
            return "Need at least 7 days of data"

        avg = statistics.mean(daily_costs[:-1])
        latest = daily_costs[-1]
        change_pct = ((latest - avg) / avg * 100) if avg > 0 else 0

        prompt = f"""Analyze this cost anomaly:

Service: {service_name}
30-day average daily cost: ${avg:.2f}
Today's cost: ${latest:.2f}
Change: {change_pct:+.1f}%

If anomaly (>20% increase), provide:
1. Likely causes
2. Investigation steps (AWS CLI commands)
3. Immediate actions to reduce cost
4. Whether this needs immediate escalation"""

        if abs(change_pct) > 20:
            return self.llm(prompt)
        return f"Normal: {change_pct:+.1f}% change (within 20% threshold)"

    def generate_finops_report(self, account_id: str, period: str,
                               cost_breakdown: dict, savings_found: float) -> str:
        prompt = f"""Generate a FinOps monthly report:

AWS Account: {account_id}
Period: {period}
Total spend: ${sum(cost_breakdown.values()):,.2f}
Breakdown by service: {json.dumps(cost_breakdown, indent=2)}
Savings opportunities identified: ${savings_found:,.2f}

Write a report with:
1. Executive Summary
2. Spend by Service (top 5)
3. Cost Trends
4. Savings Achieved This Month
5. Opportunities for Next Month
6. Key Recommendations

Tone: professional, data-driven, action-oriented"""

        return self.llm(prompt)


# Sample cost data structure
SAMPLE_COST_DATA = {
    "total_monthly": 12450.30,
    "by_service": {
        "EC2": 6200.00,
        "RDS": 2100.00,
        "EKS": 1800.00,
        "S3": 450.30,
        "CloudFront": 300.00,
        "Other": 1600.00
    },
    "idle_resources": {
        "stopped_ec2_with_ebs": 12,
        "unattached_ebs_volumes": 8,
        "unused_elastic_ips": 5,
        "old_snapshots": 234
    },
    "rightsizing_opportunities": {
        "oversized_instances": 15,
        "potential_savings": 1200.00
    }
}
```

---

## Hands-On Lab

```bash
#!/bin/bash
# Lab: AI Security + FinOps Quick Wins

echo "=== Security Lab: Dockerfile Scan ==="
cat > /tmp/test.Dockerfile << 'EOF'
FROM ubuntu:latest
RUN apt-get install python3 -y
COPY . /app
ENV DB_PASSWORD=mysecretpassword123
ENV API_KEY=abc123xyz789
RUN pip3 install -r /app/requirements.txt
CMD python3 /app/app.py
EOF

echo "Scanning Dockerfile for security issues..."
curl -s http://localhost:11434/api/generate \
  -d "$(python3 -c "
import json
with open('/tmp/test.Dockerfile') as f:
    content = f.read()
print(json.dumps({
    'model': 'phi3:mini',
    'stream': False,
    'prompt': f'Security audit this Dockerfile. List each issue with: Issue, Severity, Fix:\n{content}'
}))")" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"

echo ""
echo "=== FinOps Lab: Cost Analysis ==="
curl -s http://localhost:11434/api/generate \
  -d '{"model":"phi3:mini","stream":false,"prompt":"AWS bill shows EC2=$6200, RDS=$2100, EKS=$1800 this month. Identify top 3 cost optimization actions with estimated savings."}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
```

---

## Interview Questions

### 🎯 Security Questions
```
Q1. How does AI improve traditional SAST (Static Analysis)?
    Expected: Understands context and intent, fewer false positives,
    explains why something is vulnerable, suggests specific fixes

Q2. What are the OWASP Top 10 and how can AI help detect them?
    Expected: Injection, Broken Auth, XSS, IDOR, Misconfig, etc.;
    AI can scan code for patterns, explain vulnerabilities in context

Q3. How would you use AI to generate security test cases?
    Expected: Describe the endpoint/function to AI, ask for
    positive and negative test cases, edge cases, injection attempts

Q4. What is shift-left security and how does AI enable it?
    Expected: Security earlier in pipeline (code review, not prod);
    AI makes it feasible by automating review at coding time

Q5. How do you prevent AI from generating insecure code?
    Expected: Security rules in system prompt, output validation,
    scan AI-generated code same as human code, prefer vetted patterns
```

### 🎯 FinOps Questions
```
Q6. What is FinOps and how does AI improve cloud cost management?
    Expected: Financial operations for cloud; AI detects anomalies,
    recommends rightsizing, predicts costs, finds waste automatically

Q7. How would you build an AI-powered cost anomaly detector?
    Expected: Baseline historical daily costs, use ML for anomaly
    detection (IsolationForest or Z-score), trigger when >X% deviation,
    LLM explains likely cause and remediation

Q8. What is rightsizing and how does AI help?
    Expected: Matching instance size to actual workload;
    AI analyzes CPU/memory metrics, recommends smaller type,
    estimates savings, flags when usage is unpredictable

Q9. How would you implement AI-driven tagging compliance?
    Expected: Scan all resources for required tags, LLM suggests
    appropriate tag values from resource names/types,
    enforce via policy with auto-remediation

Q10. What cost optimization should be checked first in a
     typical AWS account?
     Expected: Idle/stopped EC2 with EBS, unattached volumes,
     unused Elastic IPs, oversized instances, old snapshots,
     Reserved Instance opportunities
```

### 🎯 Advanced Combined
```
Q11. How would you build a unified DevSecFinOps AI platform?
     Expected: Security scanning at commit + build time,
     cost analysis at IaC planning stage,
     runtime threat detection with cost-aware remediation,
     unified dashboard with risk + cost metrics

Q12. How does AI help with compliance automation?
     Expected: Map controls to code checks, auto-generate
     evidence for audits, continuously monitor drift,
     LLM explains compliance gaps in plain language

Q13. What is the relationship between security and cost in cloud?
     Expected: Security tools cost money (WAF, GuardDuty);
     security incidents cost more (breach, downtime);
     over-provisioned resources waste money;
     AI helps optimize both simultaneously

Q14. How would you use AI to automate SOC2 compliance reporting?
     Expected: Continuously check controls with automated scanners,
     LLM generates evidence narratives from check results,
     auto-generate control documentation,
     flag non-compliant resources for remediation

Q15. Describe an AI-powered security incident response workflow.
     Expected: SIEM alert → AI enriches with threat intel →
     AI triages severity → auto-contains if safe (block IP) →
     AI generates investigation steps → human reviews →
     AI drafts incident report → post-mortem automation
```

---
*Original content | Free to use | No copyright restrictions*

---
---
---

# Class 10 — Capstone Project & Future of AI in DevOps
> Original Content | No Copyright | Free to Use and Share

---

## 📌 Capstone Project: AI-Powered DevOps Platform

### Project Overview

```
┌─────────────────────────────────────────────────────────────────┐
│          CAPSTONE: AI DEVOPS AUTOMATION PLATFORM                 │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   CI/CD AI   │  │  INFRA AI    │  │  OPS AI      │           │
│  │              │  │              │  │              │           │
│  │ PR Review    │  │ IaC Scan     │  │ Incident Bot │           │
│  │ Test Gen     │  │ Cost Opt.    │  │ Alert Corr.  │           │
│  │ Deploy Risk  │  │ Drift Det.   │  │ Auto-heal    │           │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
│         │                 │                  │                   │
│         └─────────────────┴──────────────────┘                   │
│                           │                                      │
│               ┌───────────▼───────────┐                          │
│               │  UNIFIED AI PLATFORM  │                          │
│               │  REST API + WebUI     │                          │
│               │  Audit Log            │                          │
│               │  Multi-model Router   │                          │
│               └───────────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

### Capstone Implementation

```python
#!/usr/bin/env python3
"""
Capstone: AI DevOps Platform
A unified platform integrating all course concepts
"""

import json
import time
import uuid
from datetime import datetime
from typing import Callable, Dict, Optional


class AIDevOpsPlatform:
    """
    Unified AI DevOps Platform
    Integrates: CI/CD AI, Infrastructure AI, and Operations AI
    """

    def __init__(self, llm_func: Callable, config: Optional[Dict] = None):
        self.llm = llm_func
        self.config = config or {}
        self.audit_log = []
        self.incident_tracker = {}
        self.metrics = {
            "requests_total": 0,
            "incidents_handled": 0,
            "cost_saved": 0.0,
            "security_issues_found": 0
        }

    def _log_action(self, action: str, input_data: str, output: str, duration_ms: float):
        entry = {
            "id": str(uuid.uuid4())[:8],
            "timestamp": datetime.utcnow().isoformat(),
            "action": action,
            "input_preview": str(input_data)[:100],
            "output_preview": str(output)[:100],
            "duration_ms": duration_ms
        }
        self.audit_log.append(entry)
        self.metrics["requests_total"] += 1

    def _timed_llm(self, prompt: str, action: str, input_data: str = "") -> str:
        start = time.time()
        result = self.llm(prompt)
        duration = (time.time() - start) * 1000
        self._log_action(action, input_data, result, duration)
        return result

    # ── CI/CD Intelligence ────────────────────────────────────

    def review_pull_request(self, pr_diff: str, pr_description: str = "") -> dict:
        """AI-powered PR review"""
        prompt = f"""DevOps PR Review:
Description: {pr_description}
Diff:
{pr_diff[:2000]}

Review for: security, performance, correctness, test coverage.
JSON: {{"approval": "approve|request_changes", "issues": [...], "summary": "..."}}"""

        response = self._timed_llm(prompt, "pr_review", pr_diff[:50])
        try:
            return json.loads(response[response.find('{'):response.rfind('}')+1])
        except:
            return {"approval": "comment", "summary": response[:200]}

    def score_deployment_risk(self, deployment: dict) -> dict:
        """Score risk before deploying"""
        prompt = f"""Score deployment risk:
{json.dumps(deployment, indent=2)}

JSON: {{"risk_score": 1-10, "risk_level": "low|medium|high|critical",
"factors": [...], "recommendation": "deploy|delay|abort"}}"""

        response = self._timed_llm(prompt, "deployment_risk", str(deployment)[:50])
        try:
            return json.loads(response[response.find('{'):response.rfind('}')+1])
        except:
            return {"risk_level": "medium", "recommendation": "review"}

    # ── Infrastructure Intelligence ───────────────────────────

    def scan_iac(self, code: str, code_type: str = "terraform") -> list:
        """Scan IaC for security and cost issues"""
        prompt = f"""Security + cost audit of this {code_type}:
{code[:2000]}

JSON array: [{{"issue": "...", "type": "security|cost", "severity": "critical|high|medium|low", "fix": "..."}}]"""

        response = self._timed_llm(prompt, "iac_scan", code[:50])
        self.metrics["security_issues_found"] += 1
        try:
            start = response.find('[')
            end = response.rfind(']') + 1
            return json.loads(response[start:end])
        except:
            return [{"issue": "Unable to parse", "severity": "medium"}]

    def optimize_costs(self, cost_report: dict) -> str:
        """Generate cost optimization recommendations"""
        prompt = f"""FinOps cost optimization:
{json.dumps(cost_report, indent=2)}

Provide top 5 specific savings recommendations with $ impact."""
        return self._timed_llm(prompt, "cost_optimization", str(cost_report)[:50])

    # ── Operations Intelligence ───────────────────────────────

    def handle_incident(self, alert: dict) -> dict:
        """Full incident response workflow"""
        incident_id = f"INC-{len(self.incident_tracker)+1:04d}"

        prompt = f"""Incident Response:
Alert: {json.dumps(alert)}

JSON: {{"severity": "P1|P2|P3", "assessment": "...",
"root_cause": "...", "immediate_actions": [...], "commands": [...]}}"""

        response = self._timed_llm(prompt, "incident_response", str(alert)[:50])
        self.metrics["incidents_handled"] += 1

        try:
            analysis = json.loads(response[response.find('{'):response.rfind('}')+1])
        except:
            analysis = {"severity": "P2", "assessment": response[:200]}

        incident = {"id": incident_id, "alert": alert, "analysis": analysis,
                   "created_at": datetime.utcnow().isoformat()}
        self.incident_tracker[incident_id] = incident
        return incident

    def analyze_logs(self, logs: str, service: str = "unknown") -> str:
        """AI log analysis"""
        prompt = f"""Analyze {service} logs. Find errors, root cause, fix:
{logs[:2000]}"""
        return self._timed_llm(prompt, "log_analysis", logs[:50])

    # ── Platform Management ───────────────────────────────────

    def get_dashboard(self) -> dict:
        """Platform metrics dashboard"""
        return {
            "platform": "AI DevOps Platform",
            "uptime": "Active",
            "metrics": self.metrics,
            "recent_actions": self.audit_log[-5:],
            "open_incidents": len([i for i in self.incident_tracker.values()
                                  if i.get("status") != "resolved"])
        }

    def generate_weekly_report(self) -> str:
        """Generate AI weekly operations report"""
        dashboard = self.get_dashboard()
        prompt = f"""Generate weekly DevOps AI platform report:
Metrics: {json.dumps(dashboard['metrics'])}
Actions taken: {len(self.audit_log)}
Open incidents: {dashboard['open_incidents']}

Write: executive summary, key wins, issues, next week focus"""
        return self._timed_llm(prompt, "weekly_report", "")


# Complete Integration Example
def run_capstone_demo(llm_func: Callable):
    """Run complete platform demo"""
    platform = AIDevOpsPlatform(llm_func)

    print("=== AI DevOps Platform Demo ===\n")

    # 1. PR Review
    print("1. Reviewing Pull Request...")
    pr_result = platform.review_pull_request(
        pr_diff="- replicas: 1\n+ replicas: 3\n+ resources:\n+   limits:\n+     memory: 512Mi",
        pr_description="Scale up payment service for Black Friday"
    )
    print(f"   Approval: {pr_result.get('approval')}")

    # 2. Deployment Risk
    print("\n2. Scoring Deployment Risk...")
    risk = platform.score_deployment_risk({
        "service": "payment-api",
        "environment": "production",
        "change_type": "scale",
        "time": "14:30 UTC Friday"
    })
    print(f"   Risk Level: {risk.get('risk_level')}")

    # 3. Incident Response
    print("\n3. Handling Incident...")
    incident = platform.handle_incident({
        "alertname": "ServiceDown",
        "service": "payment-api",
        "severity": "critical"
    })
    print(f"   Incident: {incident['id']} - {incident['analysis'].get('severity')}")

    # 4. Dashboard
    print("\n4. Platform Dashboard:")
    dash = platform.get_dashboard()
    print(json.dumps(dash["metrics"], indent=2))

    return platform
```

---

## 📌 Future of AI in DevOps

```
┌─────────────────────────────────────────────────────────────────┐
│              AI DEVOPS: 2025 → 2030 ROADMAP                     │
│                                                                  │
│  2025 (NOW)                                                      │
│  ├── AI-assisted code review and generation                     │
│  ├── AI log analysis and RCA                                    │
│  ├── AI-powered alert correlation                               │
│  └── AI cost optimization recommendations                       │
│                                                                  │
│  2026-2027                                                       │
│  ├── Fully autonomous CI/CD with AI approvals                   │
│  ├── AI predicts incidents before they occur                    │
│  ├── Self-healing infrastructure at scale                       │
│  └── Natural language infrastructure management                 │
│                                                                  │
│  2028-2030                                                       │
│  ├── AI designs optimal architecture from requirements          │
│  ├── Continuous autonomous optimization                         │
│  ├── AI writes and maintains infrastructure code                │
│  └── DevOps engineer role evolves to AI supervisor             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Complete Interview Preparation Guide

### Top 15 Questions Across All Topics

```
FUNDAMENTAL AI CONCEPTS
────────────────────────
Q1. Explain LLMs, tokens, temperature, and context window.
Q2. Difference between RAG, fine-tuning, and prompt engineering.
Q3. What is an AI agent and how does ReAct work?

PRACTICAL DEVOPS AI
────────────────────
Q4. How would you use AI to debug a CrashLoopBackOff pod?
Q5. Design an AI-powered incident response workflow.
Q6. How do you integrate AI into a CI/CD pipeline safely?
Q7. How would you build an AI log analyzer?

ARCHITECTURE
─────────────
Q8. Design an AIOps platform for a 500-service microservices environment.
Q9. Local vs cloud LLM: when to use each?
Q10. How do you ensure reliability of AI automation in production?

SECURITY & COST
────────────────
Q11. How does AI improve security scanning?
Q12. What is FinOps and how does AI automate it?
Q13. Security risks of using external LLMs with production data?

ADVANCED
─────────
Q14. How do you build a feedback loop to improve AI over time?
Q15. What is the future of the DevOps engineer role with AI?
```

### Mock Interview Scenario

```
SCENARIO: "You are a senior DevOps engineer at a startup.
The CTO wants to 'add AI to our DevOps pipeline'.
Walk me through your 90-day plan."

STRONG ANSWER STRUCTURE:
Day 1-30 (Foundation):
  - Audit current pain points (incident MTTR, manual reviews)
  - Set up local Ollama for safe experimentation
  - Implement AI PR review (lowest risk, high value)
  - Build prompt library for team

Day 31-60 (Operations AI):
  - AI-powered log analysis integrated with Grafana
  - Alert correlation to reduce noise
  - AI runbook generation for top 10 alert types
  - Cost anomaly detection on AWS

Day 61-90 (Agents & Scale):
  - Self-healing agent for known failure patterns
  - AI deployment risk scoring in CI/CD
  - Security scanning automation
  - Measure ROI: MTTR reduction, hours saved, incidents prevented
```

---

## Hands-On Final Lab

```bash
#!/bin/bash
# Final Lab: Build your AI DevOps dashboard in 5 minutes

echo "=== AI DevOps Platform - Quick Demo ==="

# Task 1: AI Incident Handler
echo "1. Incident Response Test..."
curl -s http://localhost:11434/api/generate \
  -d '{"model":"phi3:mini","stream":false,"prompt":"P1 Incident: Payment service returning 500 errors. Provide: severity, root cause hypothesis, 3 kubectl commands to run, immediate action. Be brief."}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"

echo ""
echo "2. Security Scan Test..."
DOCKERFILE_ISSUES="FROM ubuntu:latest\nRUN apt-get install python3\nENV SECRET_KEY=abc123\nCOPY . /app"
curl -s http://localhost:11434/api/generate \
  -d "$(python3 -c "import json; print(json.dumps({'model':'phi3:mini','stream':False,'prompt':'Security issues in this Dockerfile (one line each):\n$DOCKERFILE_ISSUES'}))")" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"

echo ""
echo "3. Cost Optimization Test..."
curl -s http://localhost:11434/api/generate \
  -d '{"model":"phi3:mini","stream":false,"prompt":"AWS bill: EC2=$8000, RDS=$3000, S3=$200 this month. Top 3 cost reduction actions with $ savings estimate."}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"

echo ""
echo "=== Congratulations! You have completed the AI DevOps Course ==="
```

---

## Interview Questions — Capstone Level

### 🎯 System Design
```
Q1. Design a complete AI DevOps platform from scratch.
    Expected: Data ingestion layer, AI processing (multiple models),
    action layer with safety controls, feedback loop,
    audit logging, multi-tenant, cost tracking

Q2. How would you migrate a team from traditional DevOps
    to AI-augmented DevOps in 6 months?
    Expected: Start small (PR review), build trust with wins,
    measure impact, expand use cases, train team,
    establish AI governance policies

Q3. What is the hardest part of building production AI
    systems for DevOps?
    Expected: Reliability (LLM APIs go down), consistency
    (non-deterministic outputs), trust (team adoption),
    safety (autonomous actions), cost at scale

Q4. How do you build a business case for AI DevOps investment?
    Expected: Baseline MTTR, alert volume, hours on manual tasks,
    quantify improvement, calculate ROI, reference case studies

Q5. What skills does a DevOps engineer need in the AI era?
    Expected: Prompt engineering, LLM API integration,
    AI architecture patterns, evaluating AI outputs,
     orchestrating agents, AI security and governance
```

---

```
╔══════════════════════════════════════════════════════════════╗
║           COMPLETE COURSE SUMMARY                            ║
║                                                              ║
║  Class 01: AI fundamentals — LLMs, tokens, DevOps fit       ║
║  Class 02: Prompt engineering — patterns, techniques         ║
║  Class 03: Local LLMs & APIs — Ollama, OpenAI, Claude        ║
║  Class 04: AI shell scripting — automation, CLI tools        ║
║  Class 05: Observability AI — logs, incidents, alerting      ║
║  Class 06: AIOps — correlation, prediction, CMDB            ║
║  Class 07: AI Agents — ReAct, tools, multi-agent            ║
║  Class 08: Advanced Agents — GitOps, self-healing           ║
║  Class 09: Security + FinOps — scanning, cost opt.          ║
║  Class 10: Capstone + Future of AI in DevOps                ║
║                                                              ║
║  🎓 YOU ARE NOW AN AI-POWERED DEVOPS ENGINEER!               ║
╚══════════════════════════════════════════════════════════════╝
```

---
*Original content | Free to use | No copyright restrictions*
