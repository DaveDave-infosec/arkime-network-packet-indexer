# Troubleshooting and Validation Notes

This record distinguishes observed results from planned or optional tests.

## 1. OpenSearch Authentication in Arkime Database Utility

**Symptom**

```text
malformed JSON string ... before "Unauthorized"
```

**Context**

OpenSearch 3.9.0 was installed with authenticated HTTPS on localhost. The initial database utility invocation did not include credentials.

**Diagnostic step**

```bash
sudo /opt/arkime/db/db.pl --help
```

The built-in usage displayed `--esuser <user>[:<password>]` and `--insecure`.

**Validated resolution**

```bash
sudo /opt/arkime/db/db.pl --insecure --esuser admin https://localhost:9200 info
```

The script prompted for the administrator password (not exposed as a command-line argument). Result: cluster `opensearch`, version `3.9.0`, one data node, database version `-1` before initialization.

The fresh database was initialized with:

```bash
sudo /opt/arkime/db/db.pl --insecure --esuser admin https://localhost:9200 init
```

Result: `This is a fresh Arkime install`, followed by `Creating` and `Finished`.

**Caution:** `init` can delete existing Arkime data. Use it only in a fresh or deliberately reset lab after verifying backups and scope. `--insecure` was accepted only for the self-signed local lab endpoint.

## 2. Arkime Capture Service Failed to Start

**Symptom**

```text
arkimecapture.service: failed (Result: exit-code)
```

**Inspection**

```bash
sudo systemctl status arkimecapture.service --no-pager
sudo tail -n 40 /opt/arkime/logs/capture.log
```

**Root cause**

```text
FATAL CONFIG ERROR - Couldn't stat oui file
'/opt/arkime/etc/oui.txt' with error 'No such file or directory'
```

The same log contained successful `200` responses from OpenSearch, helping isolate this as a missing local reference-data file rather than a search backend connectivity problem.

**Resolution**

```bash
sudo /opt/arkime/bin/arkime_update_geo.sh
ls -lh /opt/arkime/etc/oui.txt
sudo systemctl reset-failed arkimecapture.service && sudo systemctl start arkimecapture.service
sudo systemctl status arkimecapture.service --no-pager
```

The update utility retrieved IANA IPv4 allocation information and Wireshark manufacturer data. The `oui.txt` file existed afterward (approximately 3.1 MB). Capture then reported `active (running)`.

Missing Geo Country/ASN files were warnings, not the fatal startup cause.

## 3. Service and Resource Checks

Useful checks exercised during installation:

```bash
free -h
df -h /
sysctl vm.max_map_count
dpkg --print-architecture
sudo grep -E '^-Xm[sx]' /etc/opensearch/jvm.options
sudo systemctl status opensearch --no-pager
sudo systemctl status arkimeviewer.service --no-pager
```

Observed: AMD64 Ubuntu guest, 3.8 GiB usable memory, 3.8 GiB swap, `vm.max_map_count = 1048576`, 30 GB root filesystem with roughly 22 GB available before OpenSearch installation, and 1 GB configured JVM heap.

## 4. End-to-End Capture Validation

- Arkime Viewer loaded successfully on the Windows host over `http://192.168.119.131:8005`.
- Sessions page displayed four records: SSDP, mDNS, and TCP.
- The expanded TCP record showed four packets, zero application data bytes, VMware MAC OUI identification, and port 8005.
- PCAP export succeeded.
- A separate `curl -I http://example.com` returned HTTP `200 OK`, **but was not verified as a matching Arkime session**.

These observations validate basic capture/indexing/PCAP retrieval without overstating network visibility or threat detection.

## 5. Practical Notes

- The `OPTIONS` unset environment-variable notice in the systemd logs did not prevent the services from running.
- A running service is not sufficient evidence by itself: check logs, API responses, indexed results and packet retrieval separately.
- Do not publish live `config.ini` files containing OpenSearch Basic Auth credentials or S2S secrets.
