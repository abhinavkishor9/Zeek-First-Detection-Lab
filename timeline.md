
# Investigation Timeline — Zeek Lab 06

## Overview

This timeline records the major actions and observations from the custom Zeek detection exercise. The sequence is based on the available terminal output and log evidence; it does not represent the full duration of every investigative action.

## Investigation Timeline

| Stage | Action | Observation |
|---|---|---|
| Lab preparation | Created `~/zeek-labs/lab06` | Dedicated directory prepared for the detection exercise |
| Input preparation | Copied `beaconing-investigation.pcap` from Lab 05 | Controlled PCAP available for offline analysis |
| Detection development | Created `first-detection.zeek` | Custom notice type and destination-port condition defined |
| Initial execution | Attempted to specify the output directory using `-l` | Zeek returned an argument error |
| Execution adjustment | Created `zeek-output` and ran Zeek from that directory | Generated logs in the selected working directory |
| Connection review | Examined `conn.log` using `zeek-cut` | Eight connections to TCP/8765 observed |
| Notice review | Inspected `notice.log` | Six custom detection notices observed |
| Coverage validation | Compared connection and notice counts | Two connection records were not represented by visible notices; cause remains unresolved |

## Connection Evidence

The eight connection timestamps recorded in `conn.log` were:

    1792456236.173319
    1792456241.177993
    1792456246.182732
    1792456251.187124
    1792456256.188880
    1792456261.190142
    1792456266.191321
    1792456271.194443

All eight records showed the following:

- Origin address: `127.0.0.1`
- Responder address: `127.0.0.1`
- Destination port: `8765`
- Service: `http`
- Connection state: `SF`

## Notice Evidence

The supplied `notice.log` contained six records using the custom notice type `FirstDetection::Test_Port_Connection`.

The notice timestamps corresponded to the second through seventh connection timestamps. The first and eighth connection timestamps did not appear in the supplied notice output.

This identifies the records requiring further investigation but does not explain why the corresponding notices were absent.

## Final Assessment

The custom detection generated notices for the configured destination-port condition. The exercise demonstrates the basic workflow of writing a Zeek rule, processing an offline PCAP, and reviewing structured alerts.

The difference between eight connection records and six notices remains an unresolved validation issue. The capture contains controlled localhost traffic, so the findings demonstrate detection behavior rather than malicious intent or confirmed command-and-control activity.
