+++
tags = ['daytona', 'ai-development', 'code-execution', 'dev-environment', 'ai-agents']
---

# Run AI Code Directly: Skip the JSON Dance

## The Problem with JSON-Wrapped Code Execution

When AI agents generate code, the output is typically wrapped in JSON:

```json
{
  "language": "python",
  "code": "print('hello world')",
  "explanation": "This prints hello world"
}
```

Then your application has to:
1. Parse the JSON
2. Extract the code block
3. Validate the language
4. Handle malformed JSON when the agent hallucinates a key
5. Execute the extracted code
6. Capture stdout/stderr
7. Return results

That's 7 steps between "agent wants to run code" and "code actually runs." Each step is a failure point.

## The Direct Approach

Daytona lets AI agents run code directly — no JSON parsing, no extraction, no intermediate representation. The agent writes code to a file and executes it. That's it.

```bash
# Agent writes code
echo "print('hello world')" > /workspace/script.py

# Agent runs it directly
python3 /workspace/script.py
```

Output goes straight to stdout. Errors go to stderr. Exit codes are real exit codes.

## Why This Matters

### Speed

JSON parsing adds 50–200ms per execution round-trip. When an agent is iterating on code (write → run → fix → run), that overhead compounds. Direct execution is nearly instantaneous.

### Reliability

JSON parsing fails when:
- The agent outputs malformed JSON
- The code itself contains JSON-like strings
- The agent hallucinates extra keys or missing values
- The response is truncated mid-JSON

Direct execution has none of these problems. The agent writes code. The code runs or it doesn't.

### Complexity

JSON wrapper approach:
```
Agent → JSON → Parse → Extract → Validate → Execute → Capture → Return
```

Direct approach:
```
Agent → Write file → Execute → Done
```

## How Daytona Enables Direct Execution

### Workspace as Shared Filesystem

Daytona environments provide a persistent `/workspace` directory that both the agent and the executing code share:

```bash
# Agent writes code to shared workspace
cat > /workspace/solution.py << 'EOF'
import requests
response = requests.get('https://api.example.com/data')
print(response.json())
EOF

# Execute directly
cd /workspace && python3 solution.py
```

### Container Isolation

Each Daytona environment runs in its own container. Code execution is isolated — a runaway script can't affect the host machine or other environments:

```bash
# This runs inside the container, not on your machine
rm -rf /workspace/temp-files  # Safe — only affects the container
```

### Multi-Language Support

Daytona environments come with whatever languages you define in your devcontainer:

```json
{
  "features": {
    "ghcr.io/devcontainers/features/python:1": {},
    "ghcr.io/devcontainers/features/node:1": {},
    "ghcr.io/devcontainers/features/rust:1": {},
    "ghcr.io/devcontainers/features/go:1": {}
  }
}
```

The agent picks the right tool for the job:

```bash
# Python for data processing
python3 /workspace/analyze.py

# Node.js for API calls
node /workspace/fetch-data.js

# Rust for performance-critical code
rustc /workspace/hash.rs && /workspace/hash
```

## Real-World Example: AI Agent Iterating on Code

Here's what a typical agent workflow looks like with direct execution:

```bash
# Iteration 1: Write initial solution
cat > /workspace/solution.py << 'EOF'
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))
EOF

python3 /workspace/solution.py
# Output: 55

# Iteration 2: Optimize based on results
cat > /workspace/solution.py << 'EOF'
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a

print(fibonacci(100))
EOF

python3 /workspace/solution.py
# Output: 354224848179261915075
```

No JSON parsing between iterations. The agent writes code, runs it, reads the output, and iterates.

## Setting Up Direct Execution in Your Daytona Environment

### Basic Setup

```json
{
  "name": "Direct Execution Workspace",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/python:1": {},
    "ghcr.io/devcontainers/features/node:1": {}
  },
  "postCreateCommand": "mkdir -p /workspace && chmod 777 /workspace"
}
```

### With Resource Limits

Prevent runaway scripts from consuming all resources:

```bash
# Set memory limit (512MB)
ulimit -v 524288
python3 /workspace/script.py

# Set timeout (30 seconds)
timeout 30 python3 /workspace/script.py

# Combined
timeout 30 bash -c "ulimit -v 524288 && python3 /workspace/script.py"
```

## Security Considerations

Direct code execution is powerful. With power comes risk.

### Sandboxing

Daytona's container isolation provides the first layer. For additional security:

```bash
# Run as non-root user
useradd -m executor
su - executor -c "python3 /workspace/script.py"

# Use read-only filesystem
docker run --read-only -v /workspace:/workspace:ro python:3.11 python3 /workspace/script.py
```

### Network Restrictions

Limit what the executed code can access:

```bash
# Block network access entirely
unshare -n python3 /workspace/script.py

# Allow only specific domains
iptables -A OUTPUT -d api.example.com -j ACCEPT
iptables -A OUTPUT -j DROP
python3 /workspace/script.py
```

### Input Validation

Even with direct execution, validate the agent's inputs:

```bash
# Check file size before execution
if [ $(stat -f%z /workspace/script.py) -gt 100000 ]; then
  echo "Script too large"
  exit 1
fi
```

## Comparison: JSON-Wrapped vs. Direct Execution

| Dimension | JSON-Wrapped | Direct Execution |
|-----------|-------------|-----------------|
| **Latency** | 50–200ms per round-trip | <1ms |
| **Reliability** | Fails on malformed JSON | Fails only if code is broken |
| **Complexity** | 7-step pipeline | 2 steps (write + run) |
| **Debugging** | Parse errors are opaque | Real stderr/stdout |
| **Multi-language** | Requires per-language parser | Any language that runs in the container |
| **Security** | Moderate (JSON is mostly harmless) | Requires explicit sandboxing |

## When JSON Wrapping Still Makes Sense

JSON wrapping has its place:
- **Untrusted code**: When you don't trust the agent at all
- **Audit trails**: JSON provides structured logging of what the agent intended
- **Multi-step workflows**: When you need to inspect/modify code before execution
- **Web APIs**: When the execution result needs to pass through HTTP

For day-to-day agent workflows inside a Daytona environment, direct execution is faster, simpler, and more reliable.

## Conclusion

JSON-wrapped code execution was a reasonable default in 2023 when agents were unreliable and sandboxing was primitive. In 2026, with container-native development environments like Daytona, direct execution is the better default.

Skip the JSON dance. Write code. Run it. Read the output. Iterate.

Your agents will be faster, your code will be simpler, and your error messages will actually make sense.
