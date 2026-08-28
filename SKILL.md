---
name: skill-ble-communications
description: Design, diagnose, or optimize BLE data throughput when the answer depends on negotiated link parameters, RF conditions, host/controller behavior, and application transfer choices.
---

# BLE communications

Use this skill for BLE throughput investigations and designs, especially when a
reported speed, disconnect, stall, or parameter change needs evidence across
the stack. Do not use it for a generic BLE introduction or ordinary GATT CRUD.

## Operating rules

- Preserve user authority. Start with read-only inspection and a reproducible
  baseline. Ask before changing controller PHY, DLE, connection parameters,
  BlueZ state, firmware, or application behavior; define rollback and the
  measurement that will decide whether a change helped.
- Write requested/configured values separately from effective negotiated values
  and measured behavior. A request, a host setting, a controller capability,
  and an over-the-air result are different facts.
- Trace the complete path: application → ATT → L2CAP → Link Layer/DLE →
  connection events/PHY/RF → controller/HCI → kernel/BlueZ → application.
  Identify the direction (Central→Peripheral or Peripheral→Central), ATT
  operation, payload size, queue depth, timestamps, and failure semantics.
- Use the model and target values in
  [throughput-model.md](references/throughput-model.md) before proposing a
  tuning change. Use [throughput-audit.md](references/throughput-audit.md) to
  record evidence and classify each hypothesis. For Linux hosts, read
  [linux-bluez-observability.md](references/linux-bluez-observability.md).
- Treat PHY selection as an RF decision. 2M can be faster in clean RF, while
  1M can win at range or under interference. Forcing 1M may be a controlled
  experiment, never a root-cause conclusion by itself.
- Report measured application throughput and latency separately from theoretical
  link capacity. State which gates were inspected, tested, or deployed, and
  what remains unobserved.

## Minimum useful result

Produce a compact evidence table with the baseline, effective parameters,
traffic shape, timestamps, observed bottleneck, confidence, and next bounded
experiment. Do not call a workaround a fix until the measurement and the
causal layer agree.
