# Zeek-First-Detection-Lab
## Overview
Zeek is a network security monitoring platform that converts observed network traffic into structured events and logs. You can extend its behavior with scripts that inspect those events and identify activity matching a detection rule.

A detection typically has four parts:
- Event: The network activity Zeek observes, such as a completed connection.
- Condition: The criteria that make the activity interesting.
- Notice: An alert-like record generated when the criteria are satisfied.
- Validation: A test to confirm the detection triggers for matching traffic and does not trigger for unrelated traffic.
- 
This lab introduces custom network detection engineering with Zeek. A controlled PCAP from the previous beaconing investigation is used to develop a custom script that detects connections to TCP port `8765` and generates notices for investigation.

The exercise demonstrates how Zeek scripting can convert observed network activity into structured alerts rather than relying exclusively on manual log review.

## Lab Objectives

- Develop a custom Zeek script to identify network connections that match a defined detection condition.
- Implement a custom notice type to make selected network activity visible for investigation.
- Use Zeek connection events to evaluate destination-port information during offline PCAP analysis.
- Generate structured detection alerts and examine the context provided by each notice.
- Correlate custom detection results with connection metadata to assess whether observed activity was captured by the rule.
- Investigate discrepancies between connection records and generated notices without assuming a cause before validating the evidence.
- Understand the difference between identifying a network behavior and determining whether it represents a security threat.
- Evaluate the limitations of a basic port-based detection and identify opportunities to improve its accuracy and coverage.
  
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

## Lab Scenario

A security analyst is expanding their network monitoring capabilities by moving from manual Zeek log analysis to custom detection development. The objective is to create a rule that identifies connections to a monitored TCP port and generates structured notices that can be reviewed during an investigation.

Using a controlled PCAP containing repeated HTTP connections to TCP port `8765`, the analyst develops a Zeek script that evaluates connection metadata and raises a custom notice when the destination port matches the defined condition.

During validation, the analyst reviews the generated notices and correlates them with connection records to determine whether the detection behaves as expected.

The investigation focuses on:

- Implementing custom detection logic using Zeek scripting.
- Generating and examining notices for matching connections.
- Comparing detection alerts against the underlying network evidence.
- Identifying gaps in observed detection coverage and determining what requires further investigation.
- Assessing the limitations of port-based detection without treating a match as proof of malicious activity.

The final objective is to establish whether the detection works as intended, document any unresolved discrepancies, and identify improvements needed before considering the rule for broader monitoring use.

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

## Limitations

- The rule identifies a destination port rather than malicious intent.
- The capture contains controlled localhost traffic.
- Legitimate services using the monitored port could trigger the rule.
- Detection coverage has not been fully validated.
- The rule has not been tested against production traffic or a broader range of network scenarios.
