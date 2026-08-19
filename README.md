# 🛡️ AEGIS Core | Authorization Boundary for AI Agents

[![PyPI Version](https://img.shields.io/pypi/v/aegis-core-lortuarte-sdk.svg)](https://pypi.org/project/aegis-core-lortuarte-sdk/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)




**Local pre-execution authorization for financially consequential AI-agent tool calls.**

AEGIS Core sits between an agent's reasoning/runtime layer and high-risk tools such as payments, trades, paid APIs, and other external side effects.

It enforces budget and idempotency decisions **before execution is allowed**, including under concurrent retries.

---

## 🚨 The 1,000-Request Contention Risk

An autonomous agent does not need to be malicious to create financial damage.

Retries, asynchronous execution, runaway loops, or multiple workers can cause several individually valid tool calls to compete for the same remaining budget or repeat the same logical action.

For financially consequential tools, the critical question is:

> **What happens if an agent retries the same expensive action concurrently before the previous execution has fully resolved?**

AEGIS Core places a local authorization boundary before that external side effect.

---

## ⚡ Why AEGIS? (Local Pre-Execution Policy Enforcement)

AEGIS Core is a local Python policy gate designed to make authorization decisions before financially consequential tool execution.

It focuses on deterministic local state control rather than replacing the agent framework itself.

* **Budget Enforcement:** Block execution when the remaining authorized budget is insufficient.
* **Idempotency Control:** Prevent duplicate logical tool executions from being authorized twice.
* **Concurrency Control:** Serialize competing local authorization decisions around shared economic state.
* **Framework Agnostic:** Can sit in front of LangChain, AutoGen, CrewAI, MCP integrations, or raw Python tools.
* **Local Enforcement:** The current implementation does not require Redis, Kafka, or a remote policy service for its local mode.
* **Fail-Closed Security:** Invalid authorization/signature conditions are rejected rather than silently allowed.

> **Current scope:** AEGIS Core's tested atomicity guarantees are local and single-process. Distributed multi-process or multi-node coordination is not claimed.

---

## 📊 Benchmark Proof (Concurrent Contention Test)

In a reproducible local stress test, **1,000 authorization requests** were submitted through **100 workers** while competing for a budget sufficient for only one operation:

* ✅ **1,000** authorization requests
* ✅ **100** workers
* ✅ **1** request allowed
* ✅ **999** requests denied
* ✅ **0** overspend
* ✅ Final balance remained consistent
* ✅ Financial-loss regression matrix: **12/12 PASS**

AEGIS also includes reproducible tests for:

* ✅ Idempotency conflicts
* ✅ Tool-call cryptographic binding
* ✅ Signature failure rollback
* ✅ Concurrent limited-budget settlement
* ✅ Atomic rollback
* ✅ Exact replay handling

### Measured Local Latency

Current local benchmarks include:

* **Decision primitive:** ~0.5 µs median
* **Idempotency cache hit:** ~2.4 µs median
* **Full signed authorization:** ~46.7 µs median
* **SQLite in-memory L3 settlement:** ~147.7 µs median

These measurements describe the tested **local execution paths only**.

They do **not** represent HTTP/network round trips, distributed coordination, Stripe settlement, blockchain confirmation, or other external infrastructure latency.

---

## 📦 Quickstart & Installation

```bash
pip install aegis-core-lortuarte-sdk
```


---

## 🛠️ Proof of Concept: Pre-Execution Authorization

Wrap high-risk tools such as payments, trades, paid API calls, or irreversible writes with the AEGIS gate.

```python
from decimal import Decimal
from aegis import AegisLocalPolicyGate

# 1. Initialize the local authorization gate
aegis_gate = AegisLocalPolicyGate()

# Example: authorize this agent for $100
aegis_gate.ledger_data["agent-001"] = Decimal("100.00")


def execute_agent_payment(agent_id, tool_call_id, amount):
    # 2. Authorize spending BEFORE the external side effect
    decision = aegis_gate.evaluar_gasto(
        agent_did=agent_id,
        operation="stripe_charge",
        tool_call_id=tool_call_id,
        amount_usd=str(amount),
    )

    if decision["execution_permitted"] is True:
        # Only now execute the real external action
        # stripe.PaymentIntent.create(...)
        return "Transaction Authorized"

    return "BLOCKED: Policy denied execution"
```

The important boundary is:

```text
Agent decision
      ↓
AEGIS authorization
      ↓
ALLOW / DENY
      ↓
External tool execution
```

The financially consequential side effect happens **only after authorization succeeds**.

---

## 🧠 The Architecture (vs. LLM Gateways)

| Feature | LLM / Observability Gateways | AEGIS Core |
| :--- | :--- | :--- |
| **Primary Target** | LLM requests, prompts, tokens, tracing | **Financially consequential tool execution** |
| **Enforcement Point** | Model / API request path | **Immediately before tool execution** |
| **Budget State** | Platform dependent | **Local policy state** |
| **Idempotency** | Platform dependent | **Execution-level tool-call control** |
| **Concurrency** | Platform dependent | **Local atomic authorization boundary** |
| **Deployment** | Often remote / service based | **Local Python SDK** |
| **Current Atomicity Scope** | Platform dependent | **Single-process local execution** |

AEGIS is not intended to replace LLM gateways.

It addresses a different boundary:

> **The point where an AI agent is about to turn a decision into an economically consequential action.**

---

## 🎯 Technical Design Partners

Running AI agents that can spend money, trigger payments, execute refunds, access paid APIs, perform trades, or create other financially consequential side effects?

AEGIS Core is currently looking for **technical design partners** willing to test the authorization boundary against real agent workflows.

Priority use cases:

* 💳 Agent payments and refunds
* 💰 Treasury and credit workflows
* 🔌 Paid API / MCP tool execution
* 🔁 Retry storms and concurrent execution
* 🧾 Duplicate logical actions
* 🤖 Autonomous agent spending
* 🔐 Pre-execution authorization

The objective is not to claim distributed production readiness.

The objective is to test AEGIS against real workflows, identify where the current model breaks, fix those boundaries, and retest them.

### Install

```bash
pip install aegis-core-lortuarte-sdk
```

### Project

**AEGIS Core:** https://aegis-api.com/
