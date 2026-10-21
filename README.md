# Zeek Lab 06 — Write Your First Zeek Detection

## Overview

This lab introduces custom network detection engineering with Zeek. A controlled PCAP from the previous beaconing investigation is used to develop a custom script that detects connections to TCP port `8765` and generates notices for investigation.

The exercise demonstrates how Zeek scripting can convert observed network activity into structured alerts rather than relying exclusively on manual log review.

## Objectives

- Understand how Zeek scripts support network detection engineering.
- Define a custom Zeek notice type.
- Detect connections to a specified TCP destination port.
- Generate and review alerts in `notice.log`.
- Correlate detection output with `conn.log`.
- Investigate discrepancies between observed connections and generated notices.
- Distinguish detection success from confirmation of malicious activity.

## Lab Environment

| Component | Details |
|---|---|
| Operating system | Ubuntu 24.04 on WSL |
| Zeek version | 8.0.10 |
| Analysis mode | Offline PCAP |
| Detection language | Zeek scripting |
| Input capture | `beaconing-investigation.pcap` |
| Monitored port | TCP/8765 |
| Detection output | `notice.log` |

## Scenario

A security analyst wants to identify connections to a monitored TCP port and generate alerts for investigation. The analyst creates a Zeek script that evaluates connection metadata and raises a notice when the destination port matches the configured value.

The input capture contains eight controlled HTTP connections between localhost endpoints using TCP port `8765`. This provides repeatable traffic for testing the detection.

## Detection Logic

The script loads Zeek's notice framework and defines a custom notice type named `Test_Port_Connection`.

The `connection_state_remove` event checks the destination port of a connection. When it matches TCP/8765, the script generates a notice containing the origin address and port.

The rule identifies a network condition. It does not independently determine whether a connection is malicious.

## Execution

Run Zeek from the lab directory:

    cd ~/zeek-labs/lab06

    mkdir -p zeek-output
    cd zeek-output

    /opt/zeek/bin/zeek \
      -C \
      -r ../beaconing-investigation.pcap \
      ../first-detection.zeek

The `-C` option disables checksum validation during analysis. It does not repair or modify the PCAP.

## Review the Generated Logs

Inspect the notices:

    cat notice.log

Review connection metadata:

    /opt/zeek/bin/zeek-cut \
      ts uid id.orig_h id.orig_p id.resp_h id.resp_p service conn_state \
      < conn.log

Review notice records:

    /opt/zeek/bin/zeek-cut \
      ts uid id.orig_h id.orig_p id.resp_h id.resp_p note msg \
      < notice.log

## Findings

- Zeek generated `conn.log`, `files.log`, `http.log`, `notice.log`, and `packet_filter.log`.
- Eight connections to TCP port `8765` were observed in `conn.log`.
- All eight connections involved `127.0.0.1` as both origin and responder.
- The connections were identified as HTTP traffic with connection state `SF`.
- Six custom notices were present in `notice.log`.
- The notices used the type `FirstDetection::Test_Port_Connection`.
- Each observed notice identified the monitored destination port and included the source port.

The difference between eight connections and six notices remains unresolved. Compare connection UIDs and timestamps across both logs before claiming complete detection coverage.

## Conclusion

This lab demonstrates the basic Zeek detection workflow: develop a custom rule, process an offline PCAP, generate structured notices, and validate alerts against connection evidence.

The script generated notices for matching traffic, but the discrepancy between eight connection records and six notices requires further investigation. The results demonstrate detection of a port-based condition, not confirmed malicious activity or command-and-control behavior.

## Limitations

- The rule identifies a destination port rather than malicious intent.
- The capture contains controlled localhost traffic.
- Legitimate services using the monitored port could trigger the rule.
- Detection coverage has not been fully validated.
- The rule has not been tested against production traffic or a broader range of network scenarios.
