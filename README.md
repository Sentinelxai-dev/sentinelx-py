# sentinelx-sdk

Whether it's an AI agent, a bank transfer, or a power grid, execution is permanent.

`sentinelx-sdk` checks actions at the commit boundary, before they execute. When placed in the commit path, decisions are made server-side and every decision returns a signed receipt.

[![license](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![python](https://img.shields.io/badge/python-3.9+-blue)](https://python.org)

## Install

```bash
pip install sentinelx-sdk
```

## Quick Start

```python
from sentinelx import SentinelX, AdmissibilityError

sx = SentinelX(api_key="YOUR_API_KEY")

try:
    receipt = sx.enforce("ai.agent.action.execute", {
        "agent_id":               "agent-001",
        "action_type":            "file.write",
        "human_in_loop_required": True,
        "human_in_loop":          False,
        "action_within_scope":    True,
        "action_logged":          True,
    })
    print("ADMISSIBLE | Receipt:", receipt.receipt_hash)

except AdmissibilityError as e:
    print("INADMISSIBLE:", e.summary)
    print("Constraint:", e.constraint)
    print("Violations:", len(e.violations))
    print("Receipt hash:", e.receipt_hash)
    # Action not committed (when placed in the commit path). Receipt returned.
```

## SCADA Example

```python
receipt = sx.enforce("scada.setpoint.change", {
    "device_id":                  "rtu-456",
    "parameter":                  "voltage_setpoint",
    "operator_authorized":        True,
    "change_ticket_linked":       True,
    "change_logged":              True,
    "two_person_auth":            True,
    "rollback_procedure_defined": True,
    "action_logged":              True,
})
print("ADMISSIBLE | Receipt:", receipt.receipt_hash)
```

## Observe Mode: never throws

```python
# Always returns receipt. Never raises on INADMISSIBLE.
# Useful for logging pipelines and observe mode.
receipt = sx.evaluate("wire.transfer.execute", context)
print(receipt.verdict)  # "ADMISSIBLE" or "INADMISSIBLE"
```

## How It Works

SentinelX sits at the commit boundary between your system and execution. Before an irreversible action is committed, the service evaluates it and returns a verdict with a receipt.

- **ADMISSIBLE**: receipt returned, action may proceed
- **INADMISSIBLE**: `AdmissibilityError` raised; when placed in the commit path, the action is not committed. Receipt returned.

The commit decision is made server-side, not in the client library.

## AdmissibilityError

```python
except AdmissibilityError as e:
    e.verdict        # "INADMISSIBLE"
    e.constraint     # reason identifier
    e.violation_code # e.g. "EXAMPLE-001"
    e.violations     # violation details
    e.summary        # human-readable reason
    e.receipt_hash   # receipt hash
    e.trace_id       # unique per evaluation
    e.receipt        # full Receipt object
```

## Domain Coverage

| Domain | Example Actions |
|--------|----------------|
| AI/ML Agents | `ai.agent.action.execute`, `ml.model.deploy.production` |
| Financial | `wire.transfer.execute`, `algo.trade.execute` |
| OT/SCADA | `scada.setpoint.change`, `breaker.open.execute` |
| Grid/Energy | `load.transfer.execute`, `der.curtailment.execute.batch` |
| Cyber/RMM | `rmm.script.execute`, `rmm.privilege.escalate` |
| Healthcare | `medication.order.execute`, `patient.record.modify` |
| Mobility | `driver.payout.execute`, `surge.pricing.apply` |

## Get an API Key

```bash
curl -X POST https://enforce.sentinelx.ai/generate-key
```

Or visit [sentinelx.ai](https://sentinelx.ai).

## Links

- [sentinelx.ai](https://sentinelx.ai)
- [enforce.sentinelx.ai](https://enforce.sentinelx.ai)
- [@sentinelx/sdk on npm](https://npmjs.com/package/@sentinelx/sdk)
- [sentinelx-go on GitHub](https://github.com/xinfra-ai/sentinelx-go)

## License

Apache-2.0

Patent pending. SentinelXAI, Inc.
