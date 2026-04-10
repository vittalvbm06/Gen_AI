# Class 03 — Running LLMs Locally & Making API Calls
> Original Content | No Copyright | Free to Use and Share

---

## 📌 Table of Contents
1. [Local vs Cloud LLMs](#local-vs-cloud-llms)
2. [Running LLMs with Ollama](#running-llms-with-ollama)
3. [OpenAI API Deep Dive](#openai-api-deep-dive)
4. [Anthropic Claude API](#anthropic-claude-api)
5. [Building a DevOps AI CLI Tool](#building-a-devops-ai-cli-tool)
6. [LLM in Docker & Kubernetes](#llm-in-docker--kubernetes)
7. [Cost Management](#cost-management)
8. [Hands-On Lab](#hands-on-lab)
9. [Interview Questions](#interview-questions)

---

## 1. Local vs Cloud LLMs

```
┌──────────────────────────────────────────────────────────────────┐
│                   LOCAL vs CLOUD LLMs                            │
│                                                                  │
│  LOCAL (Ollama, LM Studio)       CLOUD (OpenAI, Anthropic)       │
│  ───────────────────────         ────────────────────────────    │
│  ✅ Private data stays local     ✅ Larger, more capable models  │
│  ✅ No API cost per token        ✅ No GPU hardware needed       │
│  ✅ Works offline                ✅ Always up-to-date models     │
│  ✅ Full control                 ✅ Scalable on demand           │
│                                                                  │
│  ❌ Requires GPU/RAM (8GB+)      ❌ Data leaves your network     │
│  ❌ Smaller models               ❌ Per-token cost               │
│  ❌ You manage updates           ❌ Rate limits                  │
│  ❌ Slower on CPU                ❌ Internet dependency          │
│                                                                  │
│  BEST FOR:                       BEST FOR:                       │
│  • Sensitive infra data          • Complex reasoning tasks       │
│  • Air-gapped environments       • High-quality code gen         │
│  • Dev/test/experimentation      • Production AI features        │
│  • Cost-sensitive workloads      • Multi-modal (image+text)      │
└──────────────────────────────────────────────────────────────────┘
```

### Hardware Requirements for Local LLMs

```
MODEL SIZE     RAM NEEDED     GPU VRAM     PERFORMANCE
──────────     ──────────     ────────     ───────────
7B params      8 GB RAM       4-6 GB       Good for coding
13B params     16 GB RAM      8 GB         Better reasoning
34B params     32 GB RAM      24 GB        Near GPT-4 quality
70B params     64 GB RAM      40+ GB       GPT-4 level
```

---

## 2. Running LLMs with Ollama

Ollama is the easiest way to run LLMs locally on Linux, Mac, and Windows.

```
┌─────────────────────────────────────────────────────────────┐
│                   OLLAMA ARCHITECTURE                        │
│                                                             │
│  CLI / API Request                                          │
│       │                                                     │
│  ┌────▼──────────────────────────┐                          │
│  │      Ollama Server            │  ← runs on localhost     │
│  │   (REST API on port 11434)    │                          │
│  └────┬──────────────────────────┘                          │
│       │                                                     │
│  ┌────▼──────────────────────────┐                          │
│  │     Model Registry            │  ← ~/.ollama/models/     │
│  │  llama3, mistral, codellama   │                          │
│  └────┬──────────────────────────┘                          │
│       │                                                     │
│  ┌────▼──────────────────────────┐                          │
│  │     Inference Engine          │  ← llama.cpp backend     │
│  │   (GPU or CPU)                │                          │
│  └───────────────────────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

### Installation & Setup

```bash
# Install Ollama (Linux)
curl -fsSL https://ollama.ai/install.sh | sh

# Start Ollama service
ollama serve

# Pull models
ollama pull llama3             # Meta's Llama 3 8B
ollama pull mistral            # Mistral 7B
ollama pull codellama          # Code-optimized Llama
ollama pull gemma2             # Google's Gemma 2
ollama pull phi3               # Microsoft's Phi-3 (small, fast)

# List downloaded models
ollama list

# Run interactive chat
ollama run llama3
ollama run codellama

# Single query (non-interactive)
ollama run mistral "Explain what a Kubernetes ingress does"

# Remove a model
ollama rm mistral
```

### Ollama REST API

```bash
# Base URL: http://localhost:11434

# List models
curl http://localhost:11434/api/tags

# Generate completion
curl http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3",
    "prompt": "Write a bash script to check disk usage",
    "stream": false,
    "options": {
      "temperature": 0.2,
      "num_predict": 500
    }
  }'

# Chat API (with conversation history)
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "codellama",
    "messages": [
      {
        "role": "system",
        "content": "You are a senior DevOps engineer."
      },
      {
        "role": "user",
        "content": "Write a health check script for nginx"
      }
    ],
    "stream": false
  }'
```

### Python Client for Ollama

```python
#!/usr/bin/env python3
"""
Ollama Python Client for DevOps Automation
Install: pip install ollama
"""

import ollama

def devops_query(question: str, model: str = "llama3") -> str:
    """Send a DevOps question to local Ollama"""
    response = ollama.chat(
        model=model,
        messages=[
            {
                "role": "system",
                "content": "You are a senior DevOps engineer. Give concise, practical answers with working code examples."
            },
            {
                "role": "user",
                "content": question
            }
        ],
        options={
            "temperature": 0.2,
            "num_predict": 1000
        }
    )
    return response["message"]["content"]


def analyze_logs(log_content: str) -> str:
    """Analyze logs using local LLM"""
    prompt = f"""Analyze these application logs:

{log_content}

Provide:
1. Summary of errors found
2. Most frequent error type
3. Likely root cause
4. Recommended fix"""

    return devops_query(prompt, model="llama3")


def generate_dockerfile(app_description: str) -> str:
    """Generate a Dockerfile using local LLM"""
    prompt = f"""Generate a production-ready Dockerfile for: {app_description}

Requirements:
- Multi-stage build to minimize image size
- Non-root user
- Health check
- Proper labels
Return ONLY the Dockerfile, no explanation."""

    return devops_query(prompt, model="codellama")


if __name__ == "__main__":
    # Example usage
    print("=== DevOps AI Assistant (Local) ===\n")

    # Example 1: Debug question
    answer = devops_query("How do I check why a Kubernetes pod is OOMKilled?")
    print("Debug Question Answer:")
    print(answer[:500])
    print("...")

    # Example 2: Generate Dockerfile
    dockerfile = generate_dockerfile("Python 3.11 FastAPI REST API with PostgreSQL")
    print("\nGenerated Dockerfile:")
    print(dockerfile[:500])
```

---

## 3. OpenAI API Deep Dive

```bash
# Install OpenAI SDK
pip install openai

# Set API key
export OPENAI_API_KEY="your-api-key-here"
```

```python
#!/usr/bin/env python3
"""
OpenAI API for DevOps
Complete guide with all key parameters
"""

from openai import OpenAI
import os

client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))


# ── Basic Completion ─────────────────────────────────────────
def basic_completion(prompt: str) -> str:
    response = client.chat.completions.create(
        model="gpt-4o-mini",          # or "gpt-4o" for better quality
        messages=[
            {"role": "system", "content": "You are a senior DevOps engineer."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.2,               # low = deterministic (good for code)
        max_tokens=1000,               # limit output length
        top_p=0.95,                    # nucleus sampling
        frequency_penalty=0.1,         # reduce repetition
        presence_penalty=0.0           # reduce topic avoidance
    )
    return response.choices[0].message.content


# ── Streaming Response ────────────────────────────────────────
def streaming_completion(prompt: str):
    """Stream response token by token (great for long outputs)"""
    stream = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.2,
        stream=True
    )
    for chunk in stream:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="", flush=True)
    print()  # newline at end


# ── Multi-turn Conversation ───────────────────────────────────
def devops_chatbot():
    """Multi-turn conversation for DevOps debugging"""
    conversation = [
        {
            "role": "system",
            "content": """You are an expert DevOps engineer.
            Help debug infrastructure issues step by step.
            Ask clarifying questions if needed."""
        }
    ]

    print("DevOps AI Debugger (type 'exit' to quit)")
    while True:
        user_input = input("\nYou: ").strip()
        if user_input.lower() == "exit":
            break

        conversation.append({"role": "user", "content": user_input})

        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=conversation,
            temperature=0.3,
            max_tokens=1000
        )

        ai_message = response.choices[0].message.content
        conversation.append({"role": "assistant", "content": ai_message})

        print(f"\nAI: {ai_message}")


# ── JSON Mode (Structured Output) ────────────────────────────
import json

def analyze_alert_structured(alert: str) -> dict:
    """Return structured JSON for automated processing"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": """You are an alert analysis system.
                Return ONLY valid JSON with these fields:
                severity (P1/P2/P3/P4),
                category (infrastructure/application/security/cost),
                summary (one sentence),
                immediate_actions (list of 3 actions),
                escalate_to (team name)"""
            },
            {"role": "user", "content": f"Analyze this alert: {alert}"}
        ],
        temperature=0.1,
        response_format={"type": "json_object"}
    )
    return json.loads(response.choices[0].message.content)


# ── Function/Tool Calling ─────────────────────────────────────
def ai_with_tools():
    """Let AI call predefined functions"""
    tools = [
        {
            "type": "function",
            "function": {
                "name": "get_pod_status",
                "description": "Get the status of a Kubernetes pod",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "namespace": {
                            "type": "string",
                            "description": "Kubernetes namespace"
                        },
                        "pod_name": {
                            "type": "string",
                            "description": "Name of the pod"
                        }
                    },
                    "required": ["namespace", "pod_name"]
                }
            }
        }
    ]

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "user", "content": "Check the status of pod web-api-abc123 in the production namespace"}
        ],
        tools=tools,
        tool_choice="auto"
    )

    # Check if AI wants to call a function
    if response.choices[0].message.tool_calls:
        tool_call = response.choices[0].message.tool_calls[0]
        function_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)
        print(f"AI wants to call: {function_name}({arguments})")
        # Here you would actually call your function


if __name__ == "__main__":
    # Test basic completion
    result = basic_completion("Write a one-liner to find top 5 memory consuming processes")
    print(result)
```

---

## 4. Anthropic Claude API

```python
#!/usr/bin/env python3
"""
Anthropic Claude API for DevOps
Install: pip install anthropic
"""

import anthropic
import os

client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))


def claude_devops(prompt: str, system: str = None) -> str:
    """Query Claude with optional system prompt"""
    system_prompt = system or "You are a senior DevOps engineer. Give practical, working answers."

    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        system=system_prompt,
        messages=[
            {"role": "user", "content": prompt}
        ],
        temperature=0.2
    )
    return message.content[0].text


def analyze_large_log(log_file_path: str) -> str:
    """Claude's 200K context is perfect for large log files"""
    with open(log_file_path, "r") as f:
        log_content = f.read()

    prompt = f"""Analyze this complete log file and provide:
1. Executive summary (2-3 sentences)
2. All ERROR entries with timestamps
3. Pattern analysis (recurring issues)
4. Root cause hypothesis
5. Top 3 recommended actions

LOG FILE:
{log_content}"""

    return claude_devops(prompt)


def generate_runbook(service: str, alert: str) -> str:
    """Generate a structured runbook for an alert"""
    return claude_devops(
        f"""Create a detailed incident runbook for:
Service: {service}
Alert: {alert}

Format:
# Runbook: [Alert Name]
## Severity: [P1/P2/P3]
## Overview
## Prerequisites
## Investigation Steps (with exact commands)
## Common Causes and Fixes
## Escalation Path
## Post-Incident Actions
""",
        system="You are an experienced SRE creating incident runbooks. Be specific with kubectl, AWS CLI, and Linux commands."
    )


def review_iac_code(code: str, code_type: str) -> str:
    """Review Infrastructure as Code"""
    return claude_devops(
        f"""Review this {code_type} code for:
- Security vulnerabilities
- Best practice violations
- Cost optimization opportunities
- Reliability concerns

Code:
{code}

Return as:
## Security Issues
## Best Practice Issues
## Cost Issues
## Reliability Issues
## Corrected Code
""",
        system="You are a cloud security and IaC expert. Be thorough but prioritize by severity."
    )


if __name__ == "__main__":
    # Example: Generate a runbook
    runbook = generate_runbook(
        service="Payment API",
        alert="HTTP 5xx Error Rate > 5% for 5 minutes"
    )
    print(runbook[:1000])
```

---

## 5. Building a DevOps AI CLI Tool

```python
#!/usr/bin/env python3
"""
devops-ai: A CLI tool that wraps LLM APIs for DevOps tasks
Usage:
  devops-ai debug "pod is crashlooping"
  devops-ai generate dockerfile "python fastapi"
  devops-ai analyze logs /var/log/app.log
  devops-ai explain "kubectl explain deployment.spec.strategy"
"""

import sys
import os
import json
import argparse
import subprocess
from pathlib import Path

# Configuration
CONFIG_FILE = Path.home() / ".devops-ai" / "config.json"
DEFAULT_MODEL = "ollama:llama3"  # or "openai:gpt-4o-mini"


def load_config():
    if CONFIG_FILE.exists():
        with open(CONFIG_FILE) as f:
            return json.load(f)
    return {"model": DEFAULT_MODEL, "temperature": 0.2}


def query_llm(prompt: str, system: str = None) -> str:
    config = load_config()
    model = config.get("model", DEFAULT_MODEL)

    if model.startswith("ollama:"):
        return query_ollama(prompt, system, model.split(":")[1])
    elif model.startswith("openai:"):
        return query_openai(prompt, system, model.split(":")[1])
    elif model.startswith("anthropic:"):
        return query_anthropic(prompt, system, model.split(":")[1])
    else:
        raise ValueError(f"Unknown model provider in: {model}")


def query_ollama(prompt, system, model):
    import urllib.request
    messages = []
    if system:
        messages.append({"role": "system", "content": system})
    messages.append({"role": "user", "content": prompt})

    data = json.dumps({"model": model, "messages": messages, "stream": False}).encode()
    req = urllib.request.Request(
        "http://localhost:11434/api/chat",
        data=data,
        headers={"Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req, timeout=120) as r:
        return json.loads(r.read())["message"]["content"]


def query_openai(prompt, system, model):
    from openai import OpenAI
    client = OpenAI()
    messages = []
    if system:
        messages.append({"role": "system", "content": system})
    messages.append({"role": "user", "content": prompt})
    resp = client.chat.completions.create(model=model, messages=messages, temperature=0.2)
    return resp.choices[0].message.content


def query_anthropic(prompt, system, model):
    import anthropic
    client = anthropic.Anthropic()
    resp = client.messages.create(
        model=model,
        max_tokens=2048,
        system=system or "You are a senior DevOps engineer.",
        messages=[{"role": "user", "content": prompt}]
    )
    return resp.content[0].text


# ── CLI Commands ──────────────────────────────────────────────

def cmd_debug(args):
    """Debug a DevOps issue"""
    issue = " ".join(args.issue)
    system = "You are a senior DevOps engineer. Debug issues step by step. Provide exact commands."
    prompt = f"""Debug this DevOps issue step by step:

Issue: {issue}

Provide:
1. Likely causes (ranked by probability)
2. Diagnostic commands to run first
3. Fix for each cause
4. How to prevent this in future"""

    print(query_llm(prompt, system))


def cmd_generate(args):
    """Generate DevOps artifacts"""
    artifact_type = args.type
    description = " ".join(args.description)

    prompts = {
        "dockerfile": f"Write a production-ready Dockerfile for: {description}. Include multi-stage build, non-root user, health check.",
        "deployment": f"Write a Kubernetes Deployment YAML for: {description}. Include resource limits, probes, security context.",
        "terraform": f"Write Terraform code for: {description}. Follow best practices with variables and outputs.",
        "ansible": f"Write an Ansible playbook for: {description}. Use idempotent tasks with proper error handling.",
        "pipeline": f"Write a GitHub Actions CI/CD pipeline for: {description}. Include lint, test, build, and deploy stages.",
    }

    prompt = prompts.get(artifact_type, f"Generate a {artifact_type} for: {description}")
    print(query_llm(prompt))


def cmd_analyze(args):
    """Analyze logs or configs"""
    if args.file == "-":
        content = sys.stdin.read()
    else:
        with open(args.file) as f:
            content = f.read()

    # Truncate if very large
    if len(content) > 50000:
        content = content[:25000] + "\n...[truncated]...\n" + content[-25000:]

    prompt = f"""Analyze this {args.type}:

{content}

Provide:
1. Summary
2. Issues found (severity: critical/high/medium/low)
3. Root cause analysis
4. Recommended actions"""

    print(query_llm(prompt))


def cmd_explain(args):
    """Explain a command or concept"""
    thing = " ".join(args.thing)
    prompt = f"""Explain this for a DevOps engineer:
{thing}

Include:
- What it does
- When to use it
- Example with real-world context
- Common pitfalls"""
    print(query_llm(prompt))


def main():
    parser = argparse.ArgumentParser(description="DevOps AI Assistant")
    subparsers = parser.add_subparsers(dest="command")

    # debug command
    debug_parser = subparsers.add_parser("debug", help="Debug a DevOps issue")
    debug_parser.add_argument("issue", nargs="+", help="Describe the issue")

    # generate command
    gen_parser = subparsers.add_parser("generate", help="Generate DevOps artifacts")
    gen_parser.add_argument("type", choices=["dockerfile", "deployment", "terraform", "ansible", "pipeline"])
    gen_parser.add_argument("description", nargs="+")

    # analyze command
    analyze_parser = subparsers.add_parser("analyze", help="Analyze logs or configs")
    analyze_parser.add_argument("file", help="File to analyze (or - for stdin)")
    analyze_parser.add_argument("--type", default="log", choices=["log", "config", "yaml", "terraform"])

    # explain command
    explain_parser = subparsers.add_parser("explain", help="Explain a concept or command")
    explain_parser.add_argument("thing", nargs="+")

    args = parser.parse_args()

    if not args.command:
        parser.print_help()
        return

    commands = {
        "debug": cmd_debug,
        "generate": cmd_generate,
        "analyze": cmd_analyze,
        "explain": cmd_explain,
    }
    commands[args.command](args)


if __name__ == "__main__":
    main()
```

---

## 6. LLM in Docker & Kubernetes

### Run Ollama in Docker

```yaml
# docker-compose.yml — Ollama with GPU support
version: "3.9"

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama-models:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    restart: unless-stopped

  # Optional: Open WebUI for browser access
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - open-webui-data:/app/backend/data
    depends_on:
      - ollama
    restart: unless-stopped

volumes:
  ollama-models:
  open-webui-data:
```

### Kubernetes Deployment for Ollama

```yaml
# ollama-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama
  namespace: ai-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ollama
  template:
    metadata:
      labels:
        app: ollama
    spec:
      containers:
        - name: ollama
          image: ollama/ollama:latest
          ports:
            - containerPort: 11434
          volumeMounts:
            - name: ollama-storage
              mountPath: /root/.ollama
          resources:
            limits:
              memory: "16Gi"
              nvidia.com/gpu: "1"
            requests:
              memory: "8Gi"
      volumes:
        - name: ollama-storage
          persistentVolumeClaim:
            claimName: ollama-pvc
      tolerations:
        - key: nvidia.com/gpu
          operator: Exists
          effect: NoSchedule
---
apiVersion: v1
kind: Service
metadata:
  name: ollama-service
  namespace: ai-tools
spec:
  selector:
    app: ollama
  ports:
    - port: 11434
      targetPort: 11434
```

---

## 7. Cost Management

```
┌────────────────────────────────────────────────────────────────┐
│                    LLM COST COMPARISON                          │
│                                                                 │
│  MODEL                INPUT COST          OUTPUT COST          │
│  ──────────────────   ──────────────────  ─────────────────    │
│  GPT-4o               $2.50/M tokens      $10/M tokens         │
│  GPT-4o-mini          $0.15/M tokens      $0.60/M tokens       │
│  Claude 3.5 Sonnet    $3.00/M tokens      $15/M tokens         │
│  Claude 3 Haiku       $0.25/M tokens      $1.25/M tokens       │
│  Ollama (local)       FREE                FREE                 │
│  Gemini 1.5 Flash     $0.075/M tokens     $0.30/M tokens       │
│                                                                 │
│  COST-SAVING STRATEGIES:                                        │
│  1. Use mini/haiku models for simple tasks                      │
│  2. Cache repeated prompts (prompt caching)                     │
│  3. Use local Ollama for dev/test                               │
│  4. Limit max_tokens in responses                               │
│  5. Batch similar requests                                      │
│  6. Set budget alerts in API dashboard                          │
└────────────────────────────────────────────────────────────────┘
```

```python
# Cost tracking wrapper
def cost_aware_query(prompt: str, model: str = "gpt-4o-mini") -> tuple[str, float]:
    """Track token usage and estimated cost"""
    COSTS = {
        "gpt-4o": {"input": 0.0025, "output": 0.01},      # per 1K tokens
        "gpt-4o-mini": {"input": 0.00015, "output": 0.0006},
        "claude-3-5-sonnet": {"input": 0.003, "output": 0.015},
        "claude-3-haiku": {"input": 0.00025, "output": 0.00125},
    }

    from openai import OpenAI
    client = OpenAI()
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        temperature=0.2
    )

    usage = response.usage
    model_cost = COSTS.get(model, {"input": 0, "output": 0})
    cost = (usage.prompt_tokens * model_cost["input"] +
            usage.completion_tokens * model_cost["output"]) / 1000

    return response.choices[0].message.content, cost
```

---

## 8. Hands-On Lab

### Lab 1: Install Ollama and Run Local LLM

```bash
#!/bin/bash
# Lab: Set up local LLM for DevOps

# Step 1: Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh
echo "Ollama installed"

# Step 2: Start Ollama in background
ollama serve &
sleep 3

# Step 3: Pull a small model (fast download)
ollama pull phi3:mini    # Only 2.3GB, runs on CPU

# Step 4: Test with DevOps questions
echo "Test 1: Kubernetes debugging"
ollama run phi3:mini "What does OOMKilled mean in Kubernetes and how do I fix it?"

echo ""
echo "Test 2: Bash scripting"
ollama run phi3:mini "Write a bash one-liner to find files modified in last 24 hours"

echo ""
echo "Test 3: Docker"
ollama run phi3:mini "What is the difference between CMD and ENTRYPOINT in Dockerfile?"

# Step 5: Test the API
echo ""
echo "Test API endpoint:"
curl -s http://localhost:11434/api/generate \
  -d '{"model":"phi3:mini","prompt":"List 5 kubectl commands for debugging","stream":false}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
```

### Lab 2: Build Your First AI DevOps Script

```python
#!/usr/bin/env python3
"""
Lab: AI-powered Kubernetes health checker
Uses local Ollama to analyze cluster health
"""

import subprocess
import json
import urllib.request


def kubectl(command: str) -> str:
    """Run kubectl command and return output"""
    try:
        result = subprocess.run(
            f"kubectl {command}",
            shell=True, capture_output=True, text=True, timeout=30
        )
        return result.stdout + result.stderr
    except Exception as e:
        return f"Error: {e}"


def ask_local_llm(prompt: str) -> str:
    """Query local Ollama"""
    data = json.dumps({
        "model": "phi3:mini",
        "prompt": prompt,
        "stream": False,
        "options": {"temperature": 0.1, "num_predict": 500}
    }).encode()

    try:
        req = urllib.request.Request(
            "http://localhost:11434/api/generate",
            data=data,
            headers={"Content-Type": "application/json"}
        )
        with urllib.request.urlopen(req, timeout=60) as r:
            return json.loads(r.read())["response"]
    except Exception as e:
        return f"[Ollama not available: {e}]"


def ai_cluster_health_check():
    """AI-powered Kubernetes cluster health check"""
    print("=== AI Kubernetes Health Check ===\n")

    # Gather data
    print("Gathering cluster data...")
    data = {
        "nodes": kubectl("get nodes -o wide"),
        "failing_pods": kubectl("get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded"),
        "events": kubectl("get events -A --sort-by=.metadata.creationTimestamp | tail -20"),
        "resource_usage": kubectl("top nodes 2>/dev/null || echo 'metrics-server not available'")
    }

    # AI Analysis
    cluster_summary = "\n".join([f"{k}:\n{v}" for k, v in data.items()])

    prompt = f"""You are an SRE. Analyze this Kubernetes cluster data and provide:
1. Overall health status (Healthy/Warning/Critical)
2. Issues found (if any)
3. Recommended actions

CLUSTER DATA:
{cluster_summary[:3000]}

Be concise and actionable."""

    print("\nAI Analysis:")
    print(ask_local_llm(prompt))


if __name__ == "__main__":
    ai_cluster_health_check()
```

---

## 9. Interview Questions

### 🎯 Basic Level

```
Q1. What is Ollama and why would a DevOps engineer use it?
    Expected: Tool to run LLMs locally; used for private data,
    offline environments, zero API cost

Q2. What is the difference between using Ollama vs OpenAI API?
    Expected: Ollama = local, free, private, smaller models;
    OpenAI = cloud, paid, more capable, needs internet

Q3. What is an API key and how should you secure it?
    Expected: Authentication credential for AI APIs;
    store in environment variables or secrets manager,
    never in code or git repos

Q4. What parameters can you tune in an LLM API call?
    Expected: model, temperature, max_tokens, top_p,
    frequency_penalty, presence_penalty, stream

Q5. What is streaming in LLM APIs and when is it useful?
    Expected: Receiving tokens as they're generated;
    useful for real-time UIs, long responses, better UX
```

### 🎯 Intermediate Level

```
Q6. How would you deploy Ollama in a Kubernetes cluster?
    Expected: Deployment with GPU toleration and node selector,
    PersistentVolume for models, Service for internal access,
    resource limits for GPU/memory

Q7. Explain how you would build cost controls for an LLM API
    in a production system.
    Expected: Track tokens per request, set max_tokens limits,
    use cheaper models for simple tasks, implement caching,
    set budget alerts in API dashboard, rate limiting per user

Q8. What is the difference between function calling and
    RAG in LLM APIs?
    Expected: Function calling = LLM decides to call your code/APIs;
    RAG = fetching relevant documents before LLM generates response

Q9. How would you implement a retry strategy for LLM API calls?
    Expected: Exponential backoff for rate limit (429) errors,
    fallback to different model or local Ollama,
    circuit breaker pattern, timeout handling

Q10. How do you evaluate which local LLM model to use for
     a DevOps task?
     Expected: Consider RAM available, task complexity,
     benchmark on sample prompts, compare accuracy vs speed,
     check if code-specific model (CodeLlama) is better
```

### 🎯 Advanced Level

```
Q11. Design a resilient LLM integration architecture for a
     DevOps automation platform.
     Expected: Primary = cloud LLM (OpenAI/Claude),
     Fallback = local Ollama, Load balancing across API keys,
     Prompt caching layer (Redis), Rate limiting, Circuit breaker,
     Async queue for non-urgent tasks

Q12. What are the security risks of sending infrastructure
     data to external LLM APIs?
     Expected: PII in logs, credentials in config files,
     internal network topology exposure, vulnerability info,
     GDPR/compliance issues; Mitigate with data scrubbing,
     private cloud LLM deployment, data classification

Q13. How would you build a multi-model LLM router that
     selects the right model per task type?
     Expected: Classify task (code gen, analysis, chat),
     route code tasks to CodeLlama/GPT-4o, analysis to Claude
     (large context), simple Q&A to cheap mini model,
     implement cost budget per task category

Q14. Explain how you would implement LLM output validation
     in a DevOps automation pipeline.
     Expected: JSON schema validation, YAML syntax checking,
     terraform validate, ansible-lint, custom regex checks,
     human approval gate for critical actions (delete, scale),
     test in staging environment first

Q15. How do you handle the non-determinism of LLMs in
     automated DevOps workflows?
     Expected: Use temperature=0, use structured output (JSON mode),
     validate output format, implement idempotent actions,
     test with multiple runs, use deterministic logic for
     critical decisions (not LLM alone)
```

---

```
╔══════════════════════════════════════════════════════════════╗
║           CLASS 03 SUMMARY                                   ║
║                                                              ║
║  ✅ Local LLMs with Ollama (free, private)                   ║
║  ✅ OpenAI API: completions, streaming, function calling      ║
║  ✅ Anthropic Claude API: large context, runbooks            ║
║  ✅ Built a DevOps AI CLI tool                               ║
║  ✅ Deployed Ollama in Docker and Kubernetes                 ║
║  ✅ Cost management strategies                               ║
║                                                              ║
║  NEXT CLASS → AI-Powered Shell Scripting & CLI Automation   ║
╚══════════════════════════════════════════════════════════════╝
```

---
*Original content | Free to use | No copyright restrictions*
