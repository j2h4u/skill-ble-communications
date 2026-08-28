# BLE throughput audit

Use this page to turn a throughput complaint into falsifiable evidence. Keep
the direction and transfer workload fixed while comparing a baseline and one
bounded change at a time. Record firmware, host/kernel/BlueZ versions,
controller identity, peer identity, RF placement, security state, payload
distribution, and test duration.

## Status vocabulary

Every row must have one status and evidence:

- `COMPLIES`: the observed/effective state meets the row's criterion.
- `DOES_NOT_COMPLY`: evidence contradicts the criterion.
- `UNKNOWN`: evidence is absent, ambiguous, or only a request/configuration.
- `NOT_APPLICABLE`: the row cannot affect this direction/workload; explain why.

Never mark a row from a default, an API return value, or a single successful
transfer alone. Include a command, trace excerpt, counter, timestamped log,
packet capture, or repeatable application measurement for every conclusion.

## Audit rows

| Layer / question | Criterion or target | Status | Required evidence |
| --- | --- | --- | --- |
| Workload | Direction, ATT operation, payload sizes, and transfer window are fixed |  | Application trace with byte counts and monotonic timestamps |
| Application | Bulk Central→Peripheral uses Write Commands where valid; bulk Peripheral→Central uses Notifications where valid |  | Operation names and completion/ack behavior |
| Application | Large data is segmented and reassembled without hidden per-chunk waits |  | Segment sizes, IDs, receiver receipts, gap timings |
| Application/host | Transmit queue stays fed through each event |  | Enqueue/dequeue timestamps, queue depth or callback trace |
| ATT | Effective MTU is known and coordinated with DLE |  | Exchange/readback from both peers or wire evidence; do not treat a library's default `mtu_size=23` as authoritative |
| L2CAP/ATT | Payload targets are 244 or 495 bytes when effective LL payload is 251 |  | Actual payload histogram and negotiated LL length |
| Link Layer | Effective max TX/RX octets are 251 when supported |  | LL length exchange, controller readback, or btmon evidence |
| Link Layer | Effective TX time supports the selected LL length (2120 µs target where applicable) |  | Controller parameter readback and peer response |
| Connection events | Effective interval is around 15 ms for this workload, unless a measured reason says otherwise |  | Negotiated parameters plus event/application timing |
| Connection events | Peripheral latency is zero when optimizing Central→Peripheral; classify it N/A for the reverse direction if it cannot limit it |  | Effective latency and transfer direction |
| Scheduling | One connection; scanning and advertising stopped if practical |  | Controller/BlueZ state and concurrent-link inventory |
| Scheduling | Reverse-direction traffic is minimized for one-way bulk transfer |  | Bidirectional byte and operation counts |
| PHY/RF | Selected PHY wins a repeatable comparison at the real placement and workload |  | PHY readback, RSSI/noise context, repeated throughput/error results |
| RF reliability | Disconnects, packet errors, retry-like stalls, and event truncation are distinguished from host starvation |  | OTA capture if available; otherwise explicitly mark HCI limits |
| Controller/HCI | Host/controller timestamps are aligned with application timestamps |  | Monotonic clock method, trace markers, offset/drift estimate |
| Kernel/BlueZ | No queue, scheduling, D-Bus, socket, or adapter errors explain the gap |  | Journald, btmon/btsnoop, queue and callback logs |
| Result | Application rate, latency, loss, and integrity meet the stated requirement |  | Repeated end-to-end receipt/checksum results and confidence interval |

## Decision discipline

1. Establish the baseline and fill every applicable row before changing a
   parameter. Keep requested/configured/effective/measured values in separate
   fields.
2. Pick the lowest layer whose evidence explains the symptom, then test one
   intervention. For example, a force-1M run can test an RF hypothesis, but it
   cannot prove that RF was the cause unless errors, event behavior, and
   application results change consistently.
3. Repeat across at least the intended RF placements and both directions when
   the product is bidirectional. A short clean sample is a gate/smoke, not
   proof of long-run reliability.
4. Close the audit only when the effective parameter, causal observation, and
   end-to-end measurement agree. Otherwise leave the row `UNKNOWN` and state
   the next observation needed.

### Wire evidence can outrank library metadata

Some Linux BLE libraries expose an uninitialized or backend-default MTU (often
`mtu_size=23`) even while the link carries larger negotiated messages. If a
btsnoop trace reassembles a 502-byte L2CAP PDU from two 251-byte HCI ACL
fragments, that PDU contains a 498-byte ATT PDU and a 495-byte attribute value
(4-byte L2CAP plus 3-byte ATT headers). This proves the larger ATT message
crossed the host-controller boundary. It does not by itself prove the exact
over-the-air LL fragmentation; classify DLE separately from its negotiated
length event or controller readback. Record both observations, label the
library field as metadata, and do not silently replace either value.
