# Troubleshooting Notes

## 1. Zeek Reported That Option `-l` Requires an Argument

### Symptom

The initial command failed with the following error:

    ERROR: Option -l requires an argument.
    Try '/opt/zeek/bin/zeek --help' for more information.

### Resolution

The output directory was created, and Zeek was run from inside it instead of using the problematic `-l` option.

    cd ~/zeek-labs/lab06

    mkdir -p zeek-output
    cd zeek-output

    /opt/zeek/bin/zeek \
      -C \
      -r ../beaconing-investigation.pcap \
      ../first-detection.zeek

This approach kept the generated logs separate from the lab files and avoided the observed command-line error.

## 2. `conn.log` Was Not Found

### Symptom

The following command failed:

    /opt/zeek/bin/zeek-cut \
      ts id.orig_h id.orig_p id.resp_h id.resp_p service conn_state \
      < zeek-output/conn.log

The shell reported that the file did not exist.

### Cause

The command was executed from inside `~/zeek-labs/lab06/zeek-output`. Adding `zeek-output/` to the filename caused the shell to look for a nested directory.

### Resolution

When the current directory is `zeek-output`, reference the log directly:

    /opt/zeek/bin/zeek-cut \
      ts id.orig_h id.orig_p id.resp_h id.resp_p service conn_state \
      < conn.log

Alternatively, return to the lab directory and reference `zeek-output/conn.log`.

## 3. `notice.log` Was Not Initially Available

### Diagnostic Steps

Verify that the input capture and script exist:

    cd ~/zeek-labs/lab06

    ls -lh first-detection.zeek beaconing-investigation.pcap

Create the output directory and run Zeek from it:

    mkdir -p zeek-output
    cd zeek-output

    /opt/zeek/bin/zeek \
      -C \
      -r ../beaconing-investigation.pcap \
      ../first-detection.zeek

Inspect the generated files:

    ls -lh

### Observed Result

The subsequent execution generated `notice.log` alongside `conn.log`, `http.log`, `files.log`, and `packet_filter.log`.

The supplied output contained six custom detection notices.

## 4. Connection Count and Notice Count Did Not Match

### Symptom

- `conn.log`: eight connections to TCP/8765.
- `notice.log`: six custom notices.

### Recommended Investigation

Extract connection details:

    /opt/zeek/bin/zeek-cut \
      uid ts id.orig_h id.orig_p id.resp_h id.resp_p \
      < conn.log

Extract notice details:

    /opt/zeek/bin/zeek-cut \
      uid ts note msg \
      < notice.log

Compare the outputs to identify which connections have corresponding notices.

The available evidence does not establish why two connections lack visible notice records. Investigate the matching records and relevant script behavior before concluding that the cause is notice suppression or event handling.

## 5. Checksum Validation

The previous beaconing lab produced TCP checksum warnings during offline analysis.

The `-C` option instructs Zeek to ignore checksum validation:

    /opt/zeek/bin/zeek \
      -C \
      -r ../beaconing-investigation.pcap \
      ../first-detection.zeek

This option does not repair checksums or modify the underlying PCAP.

