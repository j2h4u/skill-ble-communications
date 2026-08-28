# BLE Communications Skill

Evidence-first diagnosis and design for BLE throughput, stalls, disconnects,
and transfer reliability.

This skill follows a reported symptom through the complete path:

`application → ATT → L2CAP → Link Layer/DLE → connection events → PHY/RF → controller/HCI → Linux/BlueZ → application receipt`

It separates requested, configured, negotiated/effective, and measured values,
then turns competing explanations into bounded, reproducible experiments. The
goal is an end-to-end explanation supported by both link evidence and what the
application actually received—not a theoretical data-rate estimate.

`SKILL.md` is the agent workflow. The `references/` directory is a reusable
BLE wiki for modeling, auditing, and Linux/BlueZ observability.

## Install

Clone the repository into the agent skills directory:

```sh
git clone https://github.com/j2h4u/skill-ble-communications.git \
  ~/.agents/skills/skill-ble-communications
```

Or symlink an existing checkout:

```sh
ln -s /path/to/skill-ble-communications ~/.agents/skills/skill-ble-communications
```

Restart Codex after installing so it can discover the skill. See the [Codex
skills documentation](https://developers.openai.com/codex/skills).

## Use

Invoke it when a BLE answer depends on negotiated parameters, RF conditions,
controller behavior, host scheduling, or application transfer choices:

```text
$skill-ble-communications Diagnose why this BLE transfer stalls after connection.
```

Start with a read-only baseline. Record direction, ATT operation, payload
shape, queueing, timestamps, effective PHY/DLE/MTU/connection parameters,
controller and BlueZ evidence, and end-to-end receipt or integrity results.

## Important boundary

HCI captures the host/controller boundary; HCI is not an over-the-air sniffer.
Do not present HCI ACL fragmentation as proof of Link Layer fragmentation or
RF behavior. Likewise, a workaround such as preferring 1M can be a useful
controlled experiment, but it is not proof of root cause without correlated RF,
link, and application evidence.

## References

The original Punch Through reading path that started this skill:

- [BLE Throughput Optimization FAQ](https://punchthrough.com/ble-throughput-optimization-faq/)
- [ATT MTU and Throughput](https://punchthrough.com/ble-att-mtu-throughput/)
- [Connection Interval and Throughput](https://punchthrough.com/ble-connection-interval-throughput/)
- [Transfer Methods for Throughput](https://punchthrough.com/ble-transfer-methods-for-throughput/)
- [Complete six-part BLE throughput series](https://punchthrough.com/topics/ble-throughput/)

The local wiki distills those sources into reusable guidance:

- [BLE throughput model](references/throughput-model.md) — capacity chain,
  sizing targets, transfer choices, and measurement interpretation; based on
  [Punch Through’s six-part BLE throughput index](https://punchthrough.com/topics/ble-throughput/).
- [BLE throughput audit](references/throughput-audit.md) — evidence rows,
  statuses, and decision discipline for falsifiable investigations.
- [Linux and BlueZ observability](references/linux-bluez-observability.md) —
  bounded `btmon`/btsnoop, BlueZ state, timestamp alignment, and HCI limits.
