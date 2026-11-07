# BLE Communications

Evidence-first guidance for designing and diagnosing BLE throughput, stalls,
disconnects, and transfer reliability. The skill follows the path from the
application through ATT, L2CAP, link-layer and radio behavior, controller and
Linux/BlueZ, back to application receipt.

## What it does

It separates requested, configured, negotiated, and measured values and uses
bounded experiments to investigate competing explanations. Start with a
read-only baseline and collect link evidence alongside application receipt and
integrity results. [SKILL.md](SKILL.md) contains the workflow; the local
references cover the [throughput model](references/throughput-model.md),
[throughput audit](references/throughput-audit.md), and
[Linux and BlueZ observability](references/linux-bluez-observability.md).

HCI captures the host/controller boundary, not over-the-air traffic. HCI ACL
fragmentation alone does not establish link-layer fragmentation or RF behavior.
A workaround such as preferring 1M is an experiment, not a root-cause finding
without correlated RF, link, and application evidence.

## Install

Clone the full repository into the agent skills directory:

```sh
mkdir -p ~/.agents/skills
git clone https://github.com/j2h4u/skill-ble-communications.git ~/.agents/skills/ble-communications
```

If the destination already exists, inspect it first; do not overwrite it. Reload
the skill according to your agent host's instructions.

## Update

```sh
git -C ~/.agents/skills/ble-communications pull --ff-only
```

## Use

Ask an agent to use the skill when negotiated parameters, RF conditions, host
or controller behavior, or transfer choices matter. For example:

```text
$skill-ble-communications Diagnose why this BLE transfer stalls after connection.
```

The original Punch Through reading path includes the
[throughput FAQ](https://punchthrough.com/ble-throughput-optimization-faq/),
[ATT MTU](https://punchthrough.com/ble-att-mtu-throughput/),
[connection interval](https://punchthrough.com/ble-connection-interval-throughput/),
[transfer methods](https://punchthrough.com/ble-transfer-methods-for-throughput/),
and the [six-part series](https://punchthrough.com/topics/ble-throughput/).

## License

[PolyForm Noncommercial 1.0.0](LICENSE.md) permits use under its stated terms.
For commercial use, request written permission from the copyright holder.
