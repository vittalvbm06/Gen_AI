# Class 04 — AI-Powered Shell Scripting & CLI Automation
> Original Content | No Copyright | Free to Use and Share

---

## 📌 Table of Contents
1. [Why AI + Shell Scripting?](#why-ai--shell-scripting)
2. [AI Shell Script Generators](#ai-shell-script-generators)
3. [Natural Language to Shell Commands](#natural-language-to-shell-commands)
4. [AI-Enhanced Bash Patterns](#ai-enhanced-bash-patterns)
5. [Shell Script Code Review with AI](#shell-script-code-review-with-ai)
6. [Building an AI Shell Assistant](#building-an-ai-shell-assistant)
7. [Hands-On Lab](#hands-on-lab)
8. [Interview Questions](#interview-questions)

---

## 1. Why AI + Shell Scripting?

```
┌─────────────────────────────────────────────────────────────┐
│          AI SUPERPOWERS FOR SHELL SCRIPTING                  │
│                                                             │
│  BEFORE AI                    AFTER AI                      │
│  ──────────────               ──────────────                │
│  "I always forget tar flags"  "AI remembers them for me"   │
│  "Complex awk one-liners"     "Natural language → command"  │
│  "No time to write tests"     "AI generates test cases"     │
│  "Manual script review"       "Instant security audit"      │
│  "Boilerplate takes hours"    "AI generates skeleton"       │
│  "Debugging by trial/error"   "AI explains error + fix"     │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. AI Shell Script Generators

### Complete AI Script Generation Workflow

```python
#!/usr/bin/env python3
"""
AI Shell Script Generator
Generates production-ready bash scripts from natural language
"""

import os
import sys
import subprocess

SCRIPT_SYSTEM_PROMPT = """You are a senior Linux/DevOps engineer.
When asked to write bash scripts:
1. Always include: #!/bin/bash and set -euo pipefail
2. Add a usage() function and argument parsing
3. Include color-coded output (RED=error, GREEN=success, YELLOW=warn)
4. Add input validation for all parameters
5. Include logging with timestamps
6. Make scripts idempotent where possible
7. Add --dry-run flag for destructive operations
8. Return ONLY the bash script, no explanation"""

def generate_script(description: str, use_local: bool = False) -> str:
    prompt = f"Write a production-ready bash script that: {description}"

    if use_local:
        import urllib.request, json
        data = json.dumps({
            "model": "codellama",
            "messages": [
                {"role": "system", "content": SCRIPT_SYSTEM_PROMPT},
                {"role": "user", "content": prompt}
            ],
            "stream": False
        }).encode()
        req = urllib.request.Request(
            "http://localhost:11434/api/chat",
            data=data, headers={"Content-Type": "application/json"}
        )
        with urllib.request.urlopen(req, timeout=120) as r:
            return json.loads(r.read())["message"]["content"]
    else:
        # OpenAI fallback
        from openai import OpenAI
        client = OpenAI()
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": SCRIPT_SYSTEM_PROMPT},
                {"role": "user", "content": prompt}
            ],
            temperature=0.1
        )
        return resp.choices[0].message.content


def save_and_validate(script: str, filename: str) -> bool:
    """Save script and run basic syntax check"""
    with open(filename, "w") as f:
        # Remove markdown code blocks if present
        clean = script.replace("```bash", "").replace("```", "").strip()
        f.write(clean)

    os.chmod(filename, 0o755)

    # Syntax check
    result = subprocess.run(["bash", "-n", filename], capture_output=True, text=True)
    if result.returncode != 0:
        print(f"SYNTAX ERROR: {result.stderr}")
        return False

    # ShellCheck (if installed)
    check = subprocess.run(["which", "shellcheck"], capture_output=True)
    if check.returncode == 0:
        sc = subprocess.run(["shellcheck", filename], capture_output=True, text=True)
        if sc.stdout:
            print(f"ShellCheck warnings:\n{sc.stdout}")

    return True


# EXAMPLE: Generate common DevOps scripts
SCRIPT_TASKS = [
    "monitor disk usage on all mount points and send alert if any exceeds 85%",
    "backup a PostgreSQL database, compress it, and upload to S3 with retention policy",
    "check if required ports are open on a list of servers from a file",
    "rotate application logs: compress logs older than 7 days, delete logs older than 30 days",
    "health check for nginx, MySQL, and Redis - restart any that are down and log the action",
]

if __name__ == "__main__":
    task = sys.argv[1] if len(sys.argv) > 1 else SCRIPT_TASKS[0]
    print(f"Generating script for: {task}")
    script = generate_script(task, use_local=True)
    print(script)
```

---

## 3. Natural Language to Shell Commands

### The `ask` Command

```bash
#!/bin/bash
# ~/.local/bin/ask
# Usage: ask "find all python files modified today"
# Translates natural language to shell command

ASK_QUERY="$*"

# Using Ollama locally
RESPONSE=$(curl -s http://localhost:11434/api/generate \
  -d "{
    \"model\": \"codellama\",
    \"prompt\": \"Convert to a single bash command (no explanation): ${ASK_QUERY}\",
    \"stream\": false,
    \"options\": {\"temperature\": 0.0}
  }" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'].strip())")

echo ""
echo "Command: $RESPONSE"
echo ""
read -rp "Run this command? [y/N] " CONFIRM

if [[ "$CONFIRM" == "y" || "$CONFIRM" == "Y" ]]; then
    eval "$RESPONSE"
fi
```

```bash
# Example usage:
ask "find all log files larger than 100MB in /var/log"
# Output: find /var/log -name "*.log" -size +100M

ask "show top 10 processes by CPU usage"
# Output: ps aux --sort=-%cpu | head -11

ask "count lines in all python files in current directory"
# Output: find . -name "*.py" | xargs wc -l | tail -1

ask "show disk usage sorted by size for current directory"
# Output: du -sh ./* | sort -rh | head -20
```

---

## 4. AI-Enhanced Bash Patterns

### Pattern 1: AI Error Explainer

```bash
#!/bin/bash
# Wrapper that explains errors with AI

ai_explain_error() {
    local exit_code=$1
    local stderr_output=$2
    local command_run=$3

    if [ "$exit_code" -ne 0 ]; then
        echo ""
        echo "❌ Command failed with exit code $exit_code"
        echo "Asking AI to explain..."

        EXPLANATION=$(curl -s http://localhost:11434/api/generate \
          -d "$(printf '{"model":"phi3:mini","stream":false,"prompt":"Explain this shell error and how to fix it:\nCommand: %s\nError: %s\nGive: 1-sentence cause, 1-sentence fix"}' \
            "$command_run" "$stderr_output")" \
          | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])")

        echo ""
        echo "💡 AI Explanation: $EXPLANATION"
    fi
}

# Wrapper function for any command
run_with_ai_help() {
    local cmd="$*"
    STDERR_OUTPUT=$("$@" 2>&1 >/dev/null)
    local exit_code=$?
    "$@"
    if [ $exit_code -ne 0 ]; then
        ai_explain_error $exit_code "$STDERR_OUTPUT" "$cmd"
    fi
}
```

### Pattern 2: AI Script Reviewer

```bash
#!/bin/bash
# ai-review: Review any shell script for issues

SCRIPT_FILE="$1"

if [ -z "$SCRIPT_FILE" ]; then
    echo "Usage: ai-review <script.sh>"
    exit 1
fi

SCRIPT_CONTENT=$(cat "$SCRIPT_FILE")

echo "Reviewing $SCRIPT_FILE with AI..."

REVIEW=$(curl -s http://localhost:11434/api/generate \
  -d "$(python3 -c "
import json, sys
script = open('$SCRIPT_FILE').read()
payload = {
    'model': 'codellama',
    'stream': False,
    'prompt': '''Review this bash script for: security issues, error handling gaps, portability problems, and performance issues.
Format: | Issue | Severity | Line | Fix |
Script:\n''' + script
}
print(json.dumps(payload))
")" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])")

echo ""
echo "=== AI Script Review ==="
echo "$REVIEW"
```

### Pattern 3: Auto-Generated Tests

```python
#!/usr/bin/env python3
"""Generate test cases for bash scripts using AI"""

def generate_bash_tests(script_path: str) -> str:
    with open(script_path) as f:
        script = f.read()

    prompt = f"""Generate comprehensive bash test cases for this script using BATS (Bash Automated Testing System).

Script:
{script}

Create tests for:
1. Happy path (normal execution)
2. Missing arguments
3. Invalid inputs
4. File not found scenarios
5. Permission errors

Return: Complete .bats test file"""

    # Call your preferred LLM here
    return prompt  # Replace with actual LLM call


# BATS test template (AI fills in test cases)
BATS_TEMPLATE = """#!/usr/bin/env bats

# Auto-generated by AI test generator

setup() {
    # Create test fixtures
    TEST_DIR=$(mktemp -d)
    export TEST_DIR
}

teardown() {
    rm -rf "$TEST_DIR"
}

# AI generates these test cases:
# @test "script runs successfully with valid input" { ... }
# @test "script fails gracefully with missing args" { ... }
# @test "script handles missing files" { ... }
"""

print(BATS_TEMPLATE)
```

---

## 5. Shell Script Code Review with AI

### Automated Review Pipeline

```bash
#!/bin/bash
# ai-git-review: Review changed scripts before commit

# Get list of changed shell scripts
CHANGED_SCRIPTS=$(git diff --cached --name-only --diff-filter=AM | grep '\.sh$')

if [ -z "$CHANGED_SCRIPTS" ]; then
    echo "No shell scripts changed"
    exit 0
fi

ISSUES_FOUND=0

for script in $CHANGED_SCRIPTS; do
    echo "=== Reviewing: $script ==="

    # First: shellcheck
    if command -v shellcheck &>/dev/null; then
        shellcheck "$script" || ISSUES_FOUND=1
    fi

    # Then: AI review
    REVIEW=$(cat "$script" | curl -s http://localhost:11434/api/generate \
      -d @- <<PAYLOAD
{
  "model": "codellama",
  "stream": false,
  "prompt": "Review this bash script briefly. List only HIGH severity issues:\n$(cat $script)"
}
PAYLOAD
    )
    echo "$REVIEW" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
    echo ""
done

if [ $ISSUES_FOUND -gt 0 ]; then
    echo "❌ Issues found. Fix before committing."
    exit 1
fi

echo "✅ All checks passed"
```

---

## 6. Building an AI Shell Assistant

```python
#!/usr/bin/env python3
"""
Interactive AI shell assistant
Like having a senior engineer in your terminal
"""

import subprocess
import readline
import json
import urllib.request


class AIShellAssistant:
    def __init__(self, model="phi3:mini"):
        self.model = model
        self.history = []
        self.system = """You are an expert Linux/DevOps shell assistant.
        For commands: provide the exact command and a one-line explanation.
        For errors: identify cause and provide fix.
        For scripts: write production-quality bash.
        Be concise. Format commands in code blocks."""

    def ask(self, query: str) -> str:
        self.history.append({"role": "user", "content": query})

        data = json.dumps({
            "model": self.model,
            "messages": [{"role": "system", "content": self.system}] + self.history[-10:],
            "stream": False,
            "options": {"temperature": 0.2}
        }).encode()

        try:
            req = urllib.request.Request(
                "http://localhost:11434/api/chat",
                data=data, headers={"Content-Type": "application/json"}
            )
            with urllib.request.urlopen(req, timeout=90) as r:
                response = json.loads(r.read())["message"]["content"]
                self.history.append({"role": "assistant", "content": response})
                return response
        except Exception as e:
            return f"Error: {e}"

    def run(self):
        print("\n🤖 AI Shell Assistant (powered by local LLM)")
        print("Type your question or 'exit' to quit\n")

        while True:
            try:
                user_input = input("\033[1;32m❯ \033[0m").strip()
            except (EOFError, KeyboardInterrupt):
                print("\nGoodbye!")
                break

            if user_input.lower() in ("exit", "quit", "q"):
                break

            if not user_input:
                continue

            print("\033[1;34mAI: \033[0m", end="", flush=True)
            response = self.ask(user_input)
            print(response)


if __name__ == "__main__":
    assistant = AIShellAssistant()
    assistant.run()
```

---

## 7. Hands-On Lab

### Lab 1: Build the `ask` Command

```bash
#!/bin/bash
# EXERCISE: Build and test the 'ask' command

# Step 1: Create the script
mkdir -p ~/.local/bin
cat > ~/.local/bin/ask << 'SCRIPT'
#!/bin/bash
QUERY="${*:-help me}"
curl -s http://localhost:11434/api/generate \
  -d "{\"model\":\"phi3:mini\",\"prompt\":\"Single bash command for: ${QUERY}\",\"stream\":false}" \
  | python3 -c "import sys,json; r=json.load(sys.stdin)['response'].strip(); print(r)"
SCRIPT

chmod +x ~/.local/bin/ask
export PATH=$PATH:~/.local/bin

# Step 2: Test these natural language queries
ask "list all docker containers that are stopped"
ask "find files modified in last 1 hour"
ask "show nginx error logs last 50 lines"
ask "check if port 8080 is in use"
ask "kill process using port 3000"
ask "compress all log files in /var/log older than 7 days"
```

### Lab 2: AI-Powered Log Analyzer Script

```bash
#!/bin/bash
# ai-log-analyze: Analyze any log file with AI

LOG_FILE="${1:-/var/log/syslog}"
LINES="${2:-200}"

echo "Analyzing last $LINES lines of $LOG_FILE..."
LOG_SAMPLE=$(tail -n "$LINES" "$LOG_FILE" 2>/dev/null || echo "Cannot read log file")

ANALYSIS=$(curl -s -X POST http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d "$(python3 -c "
import json
log = open('$LOG_FILE').readlines()[-$LINES:]
payload = {
    'model': 'phi3:mini',
    'stream': False,
    'prompt': 'Analyze these logs. List: 1)Errors found 2)Warnings 3)Most frequent issue 4)Recommended action\n\nLOGS:\n' + ''.join(log)
}
print(json.dumps(payload))
")")

echo "$ANALYSIS" | python3 -c "import sys,json; print(json.load(sys.stdin)['response'])"
```

---

## 8. Interview Questions

### 🎯 Basic Level

```
Q1. How can AI help with shell scripting tasks?
    Expected: Generate scripts from description, explain errors,
    review for security issues, convert natural language to commands

Q2. What is ShellCheck and how does AI complement it?
    Expected: ShellCheck = static analysis for bash;
    AI adds semantic understanding, context-aware fixes, and
    can explain WHY an issue exists

Q3. How would you use AI to learn an unfamiliar shell command?
    Expected: Ask AI to explain with examples, ask for common
    use cases, ask about flags and options

Q4. What risks exist in using AI-generated shell scripts?
    Expected: Commands that delete/modify files, wrong assumptions
    about environment, security vulnerabilities, not handling errors

Q5. How do you validate an AI-generated bash script?
    Expected: bash -n for syntax check, shellcheck for lint,
    test in non-production, review destructive operations,
    add dry-run mode first
```

### 🎯 Intermediate Level

```
Q6. Design a workflow where AI helps debug a failing
    CI/CD pipeline script.
    Expected: Capture error output → send to LLM with script
    context → get explanation + fix → apply fix → test again

Q7. How would you integrate AI script generation into
    a git pre-commit hook?
    Expected: Hook triggers on .sh file changes, sends to
    AI for review, blocks commit if high-severity issues found,
    logs review to PR comment

Q8. Write a prompt to have AI generate a monitoring script
    for a specific service.
    Expected: Role=SRE, specify service name and metrics,
    specify alert thresholds, desired output format (log/slack),
    include set -euo pipefail requirement

Q9. How can AI improve the reliability of complex pipelines?
    Expected: Generate error handling for each step, suggest
    retry logic, add validation between steps, generate
    rollback procedures

Q10. Explain prompt chaining for a multi-step script generation task.
     Expected: Step 1: generate requirements list,
     Step 2: generate script skeleton,
     Step 3: fill in implementation,
     Step 4: generate test cases,
     Step 5: generate documentation
```

### 🎯 Advanced Level

```
Q11. How would you build a natural language shell interface
     that is safe for production use?
     Expected: Parse intent, classify risk level (read/write/delete),
     require confirmation for write/delete, never execute without
     human review for production, audit log all actions

Q12. Design an AI-powered script maintenance system.
     Expected: Periodically re-review existing scripts for new
     vulnerabilities, check for deprecated commands, test in
     staging, generate diff of suggested improvements, PR workflow

Q13. How do you prevent AI from generating shell commands
     that could be dangerous?
     Expected: System prompt restrictions, output validation,
     allowlist of safe operations, sandboxed testing environment,
     never use eval on AI output directly

Q14. Compare AI code generation for Python vs Bash.
     Why might Bash be riskier?
     Expected: Bash executes directly on OS, errors can cause
     data loss or system damage, less type safety, word splitting
     and globbing risks, harder to test, more environment-dependent

Q15. How would you use AI to migrate a large collection of
     legacy bash scripts to Python or Go?
     Expected: Feed each script to LLM for analysis,
     generate equivalent Python/Go, generate tests for both,
     run side-by-side comparison, validate output matches,
     gradual migration with feature flags
```

---

```
╔══════════════════════════════════════════════════════════════╗
║           CLASS 04 SUMMARY                                   ║
║                                                              ║
║  ✅ AI generates production-ready scripts from description   ║
║  ✅ Natural language → shell commands with 'ask' tool        ║
║  ✅ AI-enhanced error explanation and debugging              ║
║  ✅ Automated script review pipeline                         ║
║  ✅ Interactive AI shell assistant                           ║
║                                                              ║
║  NEXT CLASS → AI for Observability & Incident Response      ║
╚══════════════════════════════════════════════════════════════╝
```

---
*Original content | Free to use | No copyright restrictions*
