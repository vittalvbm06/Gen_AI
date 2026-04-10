# Class 01 — Introduction to AI for DevOps
> Original Content | No Copyright | Free to Use and Share

---

## 📌 Table of Contents
1. [What is AI?](#what-is-ai)
2. [AI vs ML vs DL vs GenAI](#ai-vs-ml-vs-dl-vs-genai)
3. [How AI Fits Into DevOps](#how-ai-fits-into-devops)
4. [Key AI Concepts Every DevOps Engineer Must Know](#key-ai-concepts)
5. [The AI-DevOps Landscape](#the-ai-devops-landscape)
6. [Popular AI Tools for DevOps](#popular-ai-tools-for-devops)
7. [Hands-On Lab](#hands-on-lab)
8. [Interview Questions](#interview-questions)

---

## 1. What is AI?

**Artificial Intelligence (AI)** is the simulation of human intelligence by machines. It enables machines to learn, reason, perceive, understand language, and make decisions.

```
┌─────────────────────────────────────────────────────────────┐
│                  ARTIFICIAL INTELLIGENCE                     │
│                                                             │
│   "Any technique that enables machines to mimic human       │
│    intelligence and perform tasks intelligently"            │
│                                                             │
│   Examples:                                                 │
│   • Voice assistants (Siri, Alexa)                         │
│   • Recommendation engines (Netflix, YouTube)              │
│   • Fraud detection in banking                             │
│   • Code completion (GitHub Copilot)                       │
│   • ChatGPT, Claude, Gemini                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. AI vs ML vs DL vs GenAI

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ╔══════════════════════════════════════════════════════╗       │
│   ║          ARTIFICIAL INTELLIGENCE (AI)               ║       │
│   ║                                                      ║       │
│   ║   ╔══════════════════════════════════════╗           ║       │
│   ║   ║     MACHINE LEARNING (ML)            ║           ║       │
│   ║   ║                                      ║           ║       │
│   ║   ║   ╔══════════════════════════╗       ║           ║       │
│   ║   ║   ║   DEEP LEARNING (DL)    ║       ║           ║       │
│   ║   ║   ║                          ║       ║           ║       │
│   ║   ║   ║  ╔══════════════════╗   ║       ║           ║       │
│   ║   ║   ║  ║  GENERATIVE AI  ║   ║       ║           ║       │
│   ║   ║   ║  ║  (LLMs, GPT,   ║   ║       ║           ║       │
│   ║   ║   ║  ║  Claude, etc.)  ║   ║       ║           ║       │
│   ║   ║   ║  ╚══════════════════╝   ║       ║           ║       │
│   ║   ║   ╚══════════════════════════╝       ║           ║       │
│   ║   ╚══════════════════════════════════════╝           ║       │
│   ╚══════════════════════════════════════════════════════╝       │
└──────────────────────────────────────────────────────────────────┘

AI         → Rules + Reasoning + Planning
ML         → Learn from data without explicit programming
Deep Learning → Neural networks with many layers
GenAI      → Generate new content: text, code, images, audio
```

| Term | What It Does | DevOps Example |
|------|-------------|----------------|
| **AI** | Makes decisions like a human | Alert triaging system |
| **ML** | Learns patterns from data | Anomaly detection in logs |
| **DL** | Multi-layer neural networks | Image-based incident detection |
| **LLM** | Large Language Model — understands and generates text | Code generation, root cause analysis |
| **GenAI** | Generates new content | Terraform code, runbooks, summaries |

---

## 3. How AI Fits Into DevOps

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI-POWERED DEVOPS PIPELINE                    │
│                                                                  │
│  PLAN          CODE          BUILD         TEST                  │
│  ──────        ──────        ──────        ──────                │
│  AI-driven     Copilot /     AI build      AI test               │
│  backlog       Claude code   optimization  generation            │
│  grooming      suggestion    & caching     & coverage            │
│     │              │              │              │               │
│     └──────────────┴──────────────┴──────────────┘               │
│                            │                                     │
│                    ╔═══════▼═════════╗                           │
│                    ║  CI/CD PIPELINE ║                           │
│                    ╚═══════╤═════════╝                           │
│                            │                                     │
│  RELEASE       DEPLOY      OPERATE       MONITOR                 │
│  ──────        ──────      ──────        ──────                  │
│  AI risk       Canary      AI-driven     AIOps /                 │
│  scoring       analysis    config        Anomaly                 │
│  & approvals   prediction  management    detection               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Where AI Adds Value in DevOps

```
┌──────────────────┬───────────────────────────────────────────────┐
│ DevOps Phase     │ AI Use Case                                   │
├──────────────────┼───────────────────────────────────────────────┤
│ Planning         │ AI estimates story points, detects blockers   │
│ Coding           │ Code completion, review, documentation        │
│ Building         │ Smart caching, build failure prediction       │
│ Testing          │ Auto test generation, regression detection    │
│ Deployment       │ Risk scoring, auto rollback, canary analysis  │
│ Operations       │ Auto-remediation, runbook generation          │
│ Monitoring       │ Anomaly detection, alert deduplication        │
│ Security         │ SAST, threat detection, compliance checks     │
│ Cost             │ Resource rightsizing, waste detection         │
└──────────────────┴───────────────────────────────────────────────┘
```

---

## 4. Key AI Concepts Every DevOps Engineer Must Know

### 4.1 Large Language Models (LLMs)

```
┌────────────────────────────────────────────────────────────┐
│                HOW AN LLM WORKS                            │
│                                                            │
│  INPUT TEXT         TOKENIZATION        EMBEDDING          │
│  "deploy nginx"  →  ["deploy","nginx"] → [vectors]         │
│                                            │               │
│                                    TRANSFORMER             │
│                                    ATTENTION               │
│                                    LAYERS                  │
│                                            │               │
│                                    OUTPUT TOKENS           │
│                                    "kubectl apply -f ..."  │
└────────────────────────────────────────────────────────────┘
```

### 4.2 Tokens
- LLMs process text as **tokens** (roughly 1 token ≈ 0.75 words)
- `"Kubernetes"` = 1 token; `"kubectl get pods"` = 3-4 tokens
- Costs and context limits are measured in tokens

### 4.3 Context Window
- Maximum tokens an LLM can "see" at once
- GPT-4: 128K tokens | Claude: 200K tokens
- DevOps impact: large log files or codebases may exceed limits

### 4.4 Temperature
```
Temperature 0.0  →  Deterministic, factual (use for code generation)
Temperature 0.5  →  Balanced (use for explanations)
Temperature 1.0  →  Creative, varied (use for brainstorming)
Temperature 2.0  →  Very random (rarely useful)
```

### 4.5 Inference vs Training
```
TRAINING (done by AI company)          INFERENCE (done by you)
─────────────────────────────          ────────────────────────
• Requires massive GPU clusters        • Runs on laptop or cloud
• Costs millions of dollars            • Costs cents per query
• Takes weeks/months                   • Takes milliseconds
• Happens once (pre-trained)           • Happens every API call
```

### 4.6 RAG — Retrieval-Augmented Generation
```
┌─────────────────────────────────────────────────────────────┐
│                   RAG ARCHITECTURE                           │
│                                                             │
│  User Query                                                 │
│     │                                                       │
│     ├──► Vector Search ──► Your Docs/Runbooks/Logs          │
│     │         │                                             │
│     │    Relevant Chunks                                    │
│     │         │                                             │
│     └──► LLM Prompt = Query + Relevant Chunks               │
│                   │                                         │
│              Grounded Answer (no hallucination)             │
│                                                             │
│  Use Case: "Why did prod go down?" → searches your runbooks │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. The AI-DevOps Landscape

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI TOOLS LANDSCAPE FOR DEVOPS                 │
│                                                                  │
│  ┌─────────────────┐   ┌─────────────────┐   ┌───────────────┐  │
│  │   CODE & IDE    │   │  INFRASTRUCTURE │   │  MONITORING   │  │
│  │                 │   │                 │   │               │  │
│  │ GitHub Copilot  │   │ Terraform + AI  │   │  Dynatrace    │  │
│  │ Claude          │   │ Pulumi AI       │   │  New Relic AI │  │
│  │ Cursor          │   │ Ansible + LLM   │   │  Datadog LLM  │  │
│  │ Codeium         │   │ AWS CodeWhisperer│   │  Grafana AI   │  │
│  └─────────────────┘   └─────────────────┘   └───────────────┘  │
│                                                                  │
│  ┌─────────────────┐   ┌─────────────────┐   ┌───────────────┐  │
│  │    CI/CD        │   │    SECURITY     │   │  OPERATIONS   │  │
│  │                 │   │                 │   │               │  │
│  │ Jenkins AI      │   │ Snyk AI         │   │ PagerDuty AI  │  │
│  │ GitLab AI       │   │ Wiz             │   │ ServiceNow AI │  │
│  │ CircleCI AI     │   │ Checkov + LLM   │   │ BigPanda      │  │
│  │ Harness AI      │   │ Prisma Cloud    │   │ OpsRamp       │  │
│  └─────────────────┘   └─────────────────┘   └───────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Popular AI Tools for DevOps

### Foundation Models (LLMs)

| Model | Provider | Best For |
|-------|----------|----------|
| GPT-4o | OpenAI | Code generation, analysis |
| Claude 3.5 | Anthropic | Long context, reasoning |
| Gemini 1.5 Pro | Google | Multi-modal, code |
| Llama 3 | Meta | Local/self-hosted |
| Mistral | Mistral AI | Lightweight, local |
| CodeLlama | Meta | Code-specific tasks |

### DevOps-Specific AI Tools

```
CODE ASSISTANCE          INFRA & IaC             CI/CD INTELLIGENCE
──────────────           ──────────────          ──────────────────
GitHub Copilot    ──►    Env0 + AI        ──►    Harness AI
Cursor IDE        ──►    Atlantis + LLM   ──►    GitLab Duo
Claude API        ──►    AWS Q Developer  ──►    BuildPulse
Tabnine           ──►    Azure Copilot    ──►    Aviator
```

---

## 7. Hands-On Lab

### Lab 1: Your First AI API Call (Python)

```python
#!/usr/bin/env python3
"""
Lab 01 - First AI API Call for DevOps
Purpose: Send a prompt to an LLM API and get a DevOps response
"""

import os
import json
import urllib.request

# --- Using OpenAI API ---
def call_openai(prompt: str) -> str:
    api_key = os.environ.get("OPENAI_API_KEY")
    if not api_key:
        raise ValueError("Set OPENAI_API_KEY environment variable")

    data = json.dumps({
        "model": "gpt-4o-mini",
        "messages": [
            {
                "role": "system",
                "content": "You are a senior DevOps engineer. Give concise, practical answers."
            },
            {
                "role": "user",
                "content": prompt
            }
        ],
        "max_tokens": 500,
        "temperature": 0.3
    }).encode("utf-8")

    req = urllib.request.Request(
        "https://api.openai.com/v1/chat/completions",
        data=data,
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}"
        }
    )

    with urllib.request.urlopen(req) as response:
        result = json.loads(response.read().decode())
        return result["choices"][0]["message"]["content"]


# --- Using Anthropic Claude API ---
def call_claude(prompt: str) -> str:
    api_key = os.environ.get("ANTHROPIC_API_KEY")
    if not api_key:
        raise ValueError("Set ANTHROPIC_API_KEY environment variable")

    data = json.dumps({
        "model": "claude-3-5-sonnet-20241022",
        "max_tokens": 500,
        "messages": [{"role": "user", "content": prompt}]
    }).encode("utf-8")

    req = urllib.request.Request(
        "https://api.anthropic.com/v1/messages",
        data=data,
        headers={
            "Content-Type": "application/json",
            "x-api-key": api_key,
            "anthropic-version": "2023-06-01"
        }
    )

    with urllib.request.urlopen(req) as response:
        result = json.loads(response.read().decode())
        return result["content"][0]["text"]


if __name__ == "__main__":
    devops_prompts = [
        "Write a bash script to check if nginx is running and restart it if not",
        "Explain what happens when a Kubernetes pod is in CrashLoopBackOff state",
        "Generate a Dockerfile for a Python FastAPI application",
    ]

    for prompt in devops_prompts:
        print(f"\n{'='*60}")
        print(f"PROMPT: {prompt}")
        print(f"{'='*60}")
        # Uncomment whichever API you have access to:
        # response = call_openai(prompt)
        # response = call_claude(prompt)
        # print(response)
        print("[Configure your API key to see output]")
```

### Lab 2: Compare AI Answers

```bash
#!/bin/bash
# Lab: Compare AI responses for same DevOps question
# This is a simulation — plug in real API calls

QUESTION="How do I debug a pod stuck in Pending state in Kubernetes?"

echo "Question: $QUESTION"
echo ""
echo "--- Expected AI Response Topics ---"
echo "1. kubectl describe pod <podname>"
echo "2. Check node resources: kubectl describe nodes"
echo "3. Check events: kubectl get events --sort-by=.metadata.creationTimestamp"
echo "4. Check resource requests vs available capacity"
echo "5. Check node selectors / taints / tolerations"
echo "6. Check PVC binding issues"
```

### Lab 3: Explore AI Tools (No Code Required)

```
EXPLORATION TASKS:
─────────────────
✅ Task 1: Open ChatGPT or Claude.ai
           Ask: "Write a Kubernetes deployment YAML for nginx with 3 replicas"
           Evaluate: Is the YAML syntactically correct?

✅ Task 2: Install GitHub Copilot extension in VS Code
           Open any .sh file and start typing a function
           Observe: What does it suggest?

✅ Task 3: Ask Claude or ChatGPT:
           "Review this Dockerfile and find issues:
            FROM ubuntu
            RUN apt-get install python3
            COPY . /app
            CMD python3 app.py"

✅ Task 4: Ask an LLM to explain a complex error:
           "Explain this error: OOMKilled exit code 137 in Kubernetes"

✅ Task 5: Generate a runbook using AI:
           "Write a runbook for responding to high CPU alert on a Linux server"
```

---

## 8. Interview Questions

### 🎯 Basic Level

```
Q1. What is the difference between AI, ML, and GenAI?
    Expected: AI = broad field, ML = learns from data,
    GenAI = generates new content using LLMs

Q2. What is an LLM and how does it work at a high level?
    Expected: Large Language Model trained on vast text,
    predicts next token, uses transformer architecture

Q3. What is the difference between training and inference?
    Expected: Training = building the model (done by companies),
    Inference = using the model (done by developers)

Q4. What is a token in the context of LLMs?
    Expected: Unit of text (roughly 0.75 words),
    used to measure context and cost

Q5. What is "temperature" in LLM configuration?
    Expected: Controls randomness; 0 = deterministic,
    1+ = creative
```

### 🎯 Intermediate Level

```
Q6. How would you use AI to improve your CI/CD pipeline?
    Expected: Code review, test generation, build failure
    analysis, deployment risk scoring, auto-rollback

Q7. What is RAG and why is it important for DevOps AI tools?
    Expected: Retrieval-Augmented Generation — grounds LLM
    answers in your specific docs, logs, and runbooks
    to prevent hallucination

Q8. What are the risks of using AI in a production DevOps workflow?
    Expected: Hallucinations, security (data leakage),
    cost, latency, model reliability, compliance

Q9. How does GitHub Copilot differ from using ChatGPT for coding?
    Expected: Copilot is IDE-integrated, context-aware of
    current file, real-time; ChatGPT is conversational,
    requires copy-paste

Q10. What is the context window and why does it matter?
     Expected: Max tokens model can process at once;
     matters when feeding large log files or codebases
```

### 🎯 Advanced Level

```
Q11. How would you design an AI-assisted incident response system?
     Expected: Alert → LLM analyzes metrics/logs → suggests
     RCA → fetches relevant runbooks via RAG → creates
     Jira ticket → notifies Slack

Q12. What is prompt injection and how do you protect against it
     in a DevOps automation context?
     Expected: Malicious input that hijacks AI behavior;
     protect with input sanitization, system prompts,
     output validation

Q13. How would you evaluate the output quality of an AI tool
     in a DevOps context?
     Expected: Accuracy, latency, cost-per-query, hallucination
     rate, user satisfaction, A/B testing with human eval

Q14. Explain the difference between fine-tuning a model
     vs using RAG vs prompt engineering.
     Expected:
     - Prompt Engineering: no training cost, fast iteration
     - RAG: adds your private data at query time
     - Fine-tuning: trains model on your data (expensive)

Q15. What ethical considerations apply when deploying AI
     in a DevOps context?
     Expected: Data privacy, model bias, accountability,
     explainability, security of sensitive infra data sent
     to external APIs
```

---

```
╔══════════════════════════════════════════════════════════════╗
║           CLASS 01 SUMMARY                                   ║
║                                                              ║
║  ✅ AI = making machines intelligent                         ║
║  ✅ LLMs = foundation of modern GenAI                        ║
║  ✅ DevOps uses AI across all pipeline stages                ║
║  ✅ Key concepts: tokens, temperature, context, RAG          ║
║  ✅ Tools: ChatGPT, Claude, Copilot, Harness AI              ║
║                                                              ║
║  NEXT CLASS → Prompt Engineering Masterclass                 ║
╚══════════════════════════════════════════════════════════════╝
```

---
*Original content | Free to use | No copyright restrictions*
