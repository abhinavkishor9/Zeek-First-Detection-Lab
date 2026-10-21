# Investigation Timeline
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

