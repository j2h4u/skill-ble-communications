# BLE throughput model

This is a compact working model, not a promise of a device's rate. It
summarizes the six-part [Punch Through BLE throughput series](https://punchthrough.com/topics/ble-throughput/)
(published May 11, 2026). The six source articles are:

1. [Fundamentals](https://punchthrough.com/ble-throughput-fundamentals/)
2. [1M, 2M, and Coded PHY](https://punchthrough.com/ble-phy-throughput-2/)
3. [ATT MTU, DLE, and message sizing](https://punchthrough.com/ble-att-mtu-throughput/)
4. [Connection interval and peripheral latency](https://punchthrough.com/ble-connection-interval-throughput/)
5. [Transfer methods and application design](https://punchthrough.com/ble-transfer-methods-for-throughput/)
6. [Optimization FAQ](https://punchthrough.com/ble-throughput-optimization-faq/)

## Capacity chain

For every transfer, account for each boundary:

`application bytes → ATT PDU → L2CAP PDU → LL PDU(s) → connection event → PHY/RF → controller/HCI → host/BlueZ → application receipt`

ATT and L2CAP headers are not application data. For common ATT operations,
the ATT header is 3 bytes and the L2CAP header is 4 bytes. A single ATT PDU
can be fragmented into multiple LL PDUs and possibly span connection events.
Connection events are started by the Central; the Peripheral responds with a
data or empty packet. A packet error can end the event, and a time-limited event
can defer queued data to the next interval.

## High-value targets and caveats

- Target Data Length Extension (DLE) at 251 LL payload octets. Where the stack
  exposes a transmit-time setting, 2120 µs accommodates the longest 265-byte
  LL packet at 1M PHY with security. Verify the negotiated/effective values;
  `maxTxOctets`, `maxRxOctets`, and “data length” are common names.
- Coordinate ATT MTU with DLE. A large DLE with the default MTU leaves the
  larger LL capacity unused; a large MTU without DLE causes excessive
  fragmentation. Use the highest mutually supported MTU, subject to the
  stack's limits.
- With a 251-byte LL payload, 244-byte and 495-byte application payloads pack
  one or two LL packets: `251 - 4 L2CAP - 3 ATT = 244`; `2*251 - 4 - 3 =
  495`. One extra byte (245 or 496) creates a mostly empty additional LL
  packet. These are targets for bulk transfers, not a substitute for measuring
  the actual stack and protocol.
- For bulk traffic, prefer asynchronous Notifications and Write Commands over
  Indications and Write Requests when the protocol's reliability semantics
  permit it. Link Layer delivery still applies; application acknowledgements,
  ordering, integrity, and resume behavior may still be needed.
- Keep the transmit queue fed before a connection event. A host or application
  stall can waste air time even when every negotiated parameter is ideal.
- A 15 ms connection interval is generally a strong target. Below 15 ms is not
  necessarily better: event allocation, packet-error recovery, and tail
  inefficiency can dominate. Peripheral latency only limits Central→Peripheral
  throughput; it does not inherently limit Peripheral→Central sending.
- If practical, use one connection and stop scanning/advertising during the
  bulk transfer. Multiple links, scanning, advertising, and reverse-direction
  traffic compete for half-duplex radio time. Minimize reverse traffic when
  optimizing one direction.
- Choose 2M, 1M, or Coded PHY from measured RF evidence. 2M reduces airtime in
  good RF; 1M may deliver more useful throughput when 2M's sensitivity and
  retries lose the link; Coded PHY favors range/reliability over speed.

## Measurement interpretation

Keep four values distinct:

| Value | Meaning | Example evidence |
| --- | --- | --- |
| Requested | What an API or operator asked for | MTU request, PHY request |
| Configured | What a host, firmware, or policy stores | BlueZ/controller setting |
| Effective | What peers negotiated and the controller uses | LL length exchange, connection parameters, PHY readback |
| Measured | What the application and timestamps actually show | bytes/second, event cadence, retries inferred from counters |

The series gives theoretical maxima (up to about 178 kB/s between two embedded
devices and roughly 90–100 kB/s with a mobile endpoint) under deliberate
conditions. Treat them as model sanity checks. Application throughput must
include segmentation, queue starvation, acknowledgements, scheduling, RF loss,
host overhead, and the measurement window.
