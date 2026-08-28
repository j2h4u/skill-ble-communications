# Linux and BlueZ observability

This is an evidence-first runbook for a Linux Central or Peripheral. Commands
below are intended to be read-only captures unless explicitly labelled as a
mutation. Replace angle-bracket arguments; do not paste addresses, keys, or
credential-bearing logs into reports.

## Capture the host/controller boundary

Record the adapter and software context, then capture a bounded trace around a
reproducible transfer:

```sh
uname -a
bluetoothctl show
bluetoothctl info <peer-address>
btmgmt info
sudo -n btmon --index <hci-index> -w /tmp/ble-throughput.btsnoop
```

`btmon` prints HCI traffic and can write a btsnoop-compatible capture. Start it
before connecting or negotiating parameters and stop it after the application
has emitted an unmistakable end marker. Keep the trace window bounded and
protect addresses and payloads if they contain sensitive data. Journald can
add host-side context:

```sh
journalctl -u bluetooth --since '<start>' --until '<end>' --no-pager
```

Prefer `btmon`/btsnoop and BlueZ D-Bus state over deprecated `hcitool` output.
The exact fields exposed vary by controller, kernel, BlueZ version, and
connection role.

## Establish effective, not requested, parameters

Collect the negotiated state from both peers where possible:

- PHY: read the effective TX/RX PHY after the PHY procedure; a PHY request or
  controller capability is not proof that the connection uses it.
- DLE: capture the LL length procedure or controller readback for effective
  max TX/RX octets and TX/RX time. The useful target is 251 octets and, where
  exposed, 2120 µs TX time for the longest 1M encrypted packet.
- Connection parameters: record effective interval, peripheral latency, and
  supervision timeout. A peripheral's requested values may be accepted,
  clamped, or replaced by the Central/OS.
- ATT MTU: record the exchange result at the GATT/ATT layer. A library's
  `mtu_size=23` can be an uninitialized/default Linux-backend field, so it is
  not authoritative by itself. When a btsnoop trace reassembles a 502-byte
  L2CAP PDU from two 251-byte HCI ACL fragments, it contains a 498-byte ATT PDU
  and a 495-byte attribute value; record that host-controller evidence while
  retaining the library metadata. HCI ACL fragmentation is not direct proof
  of over-the-air LL fragmentation. Establish DLE independently from its
  negotiated length event or controller readback. Do not infer an effective
  MTU from DLE alone or from one unmeasured API request.

Useful read-only views include `bluetoothctl info <peer-address>`, BlueZ
`org.bluez.Device1` properties via `busctl`, and controller-specific mgmt/HCI
events in `btmon`. If a field is not exposed, write `UNKNOWN`, not the desired
value. Preserve a timestamp and the command/version context for each result.

## Align HCI and application time

Instrument the application with monotonic timestamps at enqueue, API return,
notification/write callback, receipt, integrity check, and transfer end. Add a
unique transfer ID and byte offset to logs, not payload contents. In parallel,
run `btmon` and record its start/stop times. Correlate an unmistakable GATT
operation or connection event with the application marker, then report:

- clock source and units for each trace;
- estimated offset and drift over the capture window;
- bytes visible at the application boundary versus HCI command/event times;
- gaps caused before HCI submission, inside the controller, or after HCI
  completion.

Do not compare wall-clock log lines as if they were packet timestamps. Host
buffering, scheduler delay, controller queues, and BlueZ callbacks can shift
the apparent order.

## What HCI cannot prove

HCI/btmon is a host↔controller observation point, not an over-the-air sniffer.
HCI cannot reveal OTA CRC failures, OTA retries, or empty events reliably
without an OTA capture. It also cannot establish channel interference or
exactly why a connection event ended. Controller counters may provide clues but
are implementation-specific and incomplete. Therefore:

- do not claim “no retries” from a clean HCI trace;
- do not claim an RF root cause from a disconnect alone;
- do not claim a PHY fix because forcing 1M made one run pass;
- label OTA-dependent rows `UNKNOWN` when no sniffer or reliable controller
  telemetry exists.

Use repeated end-to-end measurements, controller/kernel logs, and (when the
decision matters) an approved OTA sniffer capture to separate RF loss from
queue starvation, scheduling, ATT waits, and BlueZ/application backpressure.
