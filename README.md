# 🤖 AI for DevOps — Complete 10-Class Course

> **100% Original Content | No Copyright | Free to Use, Share, and Modify**
> A hands-on DevOps + AI course from basics to production-ready agents.

---

## 📚 Course Structure

| Class | File | Topic | Key Skills |
|-------|------|--------|------------|
| 01 | `Class01_Introduction_to_AI_for_DevOps.md` | Intro to AI for DevOps | LLMs, tokens, AI landscape |
| 02 | `Class02_Prompt_Engineering_Masterclass.md` | Prompt Engineering | Zero/few-shot, CoT, prompt library |
| 03 | `Class03_Running_LLMs_Locally_and_API_Calls.md` | LLMs & APIs | Ollama, OpenAI, Claude API |
| 04 | `Class04_AI_Shell_Scripting_CLI_Automation.md` | AI Shell Scripting | Script gen, NL→shell, auto-review |
| 05-10 | `Class05_to_10_AI_DevOps_Complete.md` | Full Curriculum | Observability → Capstone |

---

## 🗂️ What Each Class Covers

### Class 01 — Introduction to AI for DevOps
- AI vs ML vs Deep Learning vs GenAI (visual hierarchy diagram)
- How AI fits into every DevOps pipeline stage
- Key concepts: tokens, temperature, context window, RAG
- AI tools landscape for DevOps
- **Lab**: First API call, explore AI tools
- **15 interview questions** (Basic → Advanced)

### Class 02 — Prompt Engineering Masterclass
- Anatomy of a perfect prompt (Role + Context + Task + Format + Constraints)
- Zero-shot, one-shot, few-shot, chain-of-thought, tree-of-thought
- DevOps prompt pattern library (8 patterns with examples)
- System prompts for consistent DevOps AI behavior
- **Lab**: Prompt iteration, build a prompt library, chaining
- **15 interview questions** (Basic → Advanced)

### Class 03 — Running LLMs Locally & Making API Calls
- Local vs Cloud LLMs comparison with hardware requirements
- Ollama: install, pull models, REST API, Python client
- OpenAI API: completions, streaming, JSON mode, function calling
- Anthropic Claude API: large context analysis
- Building a production DevOps AI CLI tool
- Deploying Ollama in Docker and Kubernetes
- **Lab**: Local LLM setup, AI Kubernetes health checker
- **15 interview questions** (Basic → Advanced)

### Class 04 — AI-Powered Shell Scripting & CLI Automation
- AI script generators with validation pipeline
- Natural language → shell commands (`ask` command)
- AI error explainer and script reviewer
- Auto-generated BATS test cases
- Interactive AI shell assistant
- **Lab**: Build the `ask` command, AI log analyzer
- **10 interview questions** (Basic → Advanced)

### Class 05 — AI for Observability & Incident Response
- AI-powered log analysis pipeline with pre-processing
- AI anomaly detection with statistical baselines
- Full incident response bot with auto-remediation
- Alert correlation patterns
- **Lab**: Incident response simulation
- **15 interview questions** (Basic → Advanced)

### Class 06 — AIOps: AI for IT Operations
- AIOps architecture and data sources
- AI alert correlation engine (full Python implementation)
- AI capacity forecasting
- Change risk scoring
- AI-enhanced CMDB and service mapping
- **Lab**: Alert correlation simulation
- **15 interview questions** (Basic → Advanced)

### Class 07 — AI Agents for DevOps (Hands-On)
- ReAct agent pattern (Reason → Act → Observe)
- Full DevOps agent with safe tool execution
- Multi-agent incident response system
- Agent safety guardrails
- **Lab**: Single-step agent interaction
- **10 interview questions** (All levels)

### Class 08 — AI Agents for DevOps (Advanced)
- Sequential, parallel, hierarchical, self-critique agent patterns
- GitOps agent for PR review and deployment planning
- Terraform plan analyzer
- Self-healing agent with safe remediation
- **Lab**: GitOps agent PR review simulation
- **10 interview questions** (All levels)

### Class 09 — AI for Security & FinOps
- AI security scanner: Dockerfile, K8s YAML, Terraform, secret detection
- AI-powered FinOps: cost analysis, rightsizing, anomaly detection
- Automated security reporting
- FinOps report generation
- **Lab**: Security scan + cost optimization simulation
- **15 interview questions** (Security + FinOps)

### Class 10 — Capstone Project & Future of AI in DevOps
- Complete `AIDevOpsPlatform` class integrating all course concepts
- CI/CD intelligence, Infrastructure AI, Operations AI
- AI DevOps roadmap 2025-2030
- Complete interview preparation guide
- Mock interview scenario with model answer
- **Final Lab**: 5-minute AI DevOps platform demo
- **5 system design questions**

---

## 🛠️ Prerequisites

```bash
# Python 3.10+
python3 --version

# Install Ollama (for local LLM)
curl -fsSL https://ollama.ai/install.sh | sh
ollama pull phi3:mini    # Small, runs on CPU

# Python packages
pip install openai anthropic ollama requests

# Optional
pip install shellcheck    # Script linting
```

---

## 🚀 Quick Start

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/ai-devops-course.git
cd ai-devops-course

# Start with Class 01
cat Class01_Introduction_to_AI_for_DevOps.md

# Run a quick lab (Class 04)
./ask "find all running docker containers"

# Run Ollama for local AI
ollama serve &
ollama pull phi3:mini
```

---

## 🎯 Who Is This For?

- DevOps engineers wanting to add AI to their toolkit
- SREs looking to automate incident response
- Platform engineers building AI-powered internal tools
- Students preparing for AI/DevOps interviews
- Anyone curious about practical AI in infrastructure

---

## ✅ License

Original content. No copyright. Use freely.

---

**Happy Learning! 🤖🐧🚀**
