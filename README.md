# Arkime Network Packet Indexer | SOC Home Lab

A practical network security monitoring lab deploying **Arkime 6.3.1** with **OpenSearch 3.9.0** on a dedicated **Ubuntu Server 24.04** virtual machine. The lab demonstrates live packet capture, searchable network-session metadata, and PCAP export for further forensic analysis.

## Objective

Build and validate a small network packet indexing system suitable for learning SOC network investigations. This is an infrastructure deployment and validation project, **not** a claim of detecting a real intrusion.

## Architecture

```text
VMware Workstation (NAT)
       |
 Ubuntu Server 24.04
       | ens33
       v
  Arkime Capture ---------> PCAP storage (/opt/arkime/raw)
       |
       v
OpenSearch 3.9.0 (indexed session metadata)
       |
       v
  Arkime Viewer (:8005)
       |
       v
 Browser: session search / packet inspection / PCAP export
```

See [architecture/README.md](architecture/README.md) for the architecture and visibility limitations.

## Lab Environment

| Component | Configuration |
|---|---|
| Hypervisor | VMware Workstation |
| Guest OS | Ubuntu Server 24.04 LTS |
| Resources | 2 vCPUs, 4 GB RAM, 30 GB virtual disk |
| Network mode | VMware NAT |
| Capture interface | `ens33` |
| Arkime | 6.3.1 |
| Search backend | OpenSearch 3.9.0 |
| OpenSearch JVM heap | 1 GB initial / 1 GB maximum |
| Viewer | Port 8005, accessed from Windows host |
| Administration | SSH from Windows PowerShell |

## Implementation Overview

1. Created a dedicated Ubuntu Server VM instead of modifying existing Splunk/OpenCTI systems.
2. Updated Ubuntu, checked disk, RAM, swap, architecture, and `vm.max_map_count`.
3. Installed the official Arkime Ubuntu 24.04 AMD64 package.
4. Configured Arkime Capture for `ens33` and set up its Viewer.
5. Downloaded OpenSearch's Linux x64 Debian package, checked its SHA-512 checksum, and installed it with local HTTPS authentication.
6. Confirmed OpenSearch responded on `https://localhost:9200`, then initialized Arkime's database with authenticated `db.pl`.
7. Created an Arkime Viewer administrator account, started and validated the Viewer and Capture services.
8. Inspected captured TCP/UDP sessions and successfully downloaded a PCAP from Arkime Viewer.

Credentials, keys, and working configuration containing secrets are intentionally **not** committed.

## Validation Results

| Check | Evidence / result |
|---|---|
| OpenSearch service | `active (running)` |
| OpenSearch API | Authenticated request returned version `3.9.0` |
| Arkime Viewer | Accessible from Windows on port 8005 |
| Arkime Capture | `active (running)` after resolving the OUI file issue |
| Session indexing | Arkime Sessions page displayed **4 entries** |
| Session protocols | SSDP, mDNS, and TCP observed |
| Session detail | Source/destination addresses, ports, packet and byte counts, VMware MAC OUI |
| Packet evidence | Session PCAP downloaded successfully |

A test HTTP `curl -I http://example.com` request returned `200 OK`, but **that specific HTTP exchange was not verified in Arkime's visible session results**. The evidence supports successful capture/indexing of other traffic, not confirmed indexing of that test request.

## Troubleshooting Highlights

- **Unauthorized database query:** The first `db.pl ... info` attempt returned an authentication error. Retried with `--esuser admin` (interactive password prompt) and `--insecure` for the lab's self-signed certificate.
- **Capture service crash:** Capture failed with a fatal error because `/opt/arkime/etc/oui.txt` did not exist. Ran `arkime_update_geo.sh`, confirmed the OUI file, reset the service's failed state, and restarted Capture successfully.
- **Constrained VM:** OpenSearch used a 1 GB JVM heap; the system is designed for small training captures, not sustained production retention.

See [docs/troubleshooting.md](docs/troubleshooting.md) for commands and observed outcomes.

## Evidence

Evidence to include in [screenshots/](screenshots/):

- Arkime Sessions page with the four indexed sessions.
- Expanded TCP session with packet counts, IP/ports, and MAC OUI.
- `systemctl status arkimecapture.service` showing `active (running)`.
- Optional: successful OpenSearch API response (remove tokens/credentials before publication).

The [screenshots guide](screenshots/README.md) explains safe evidence handling. Screenshots are not yet included in this repository.

## Skills Demonstrated

Linux administration • VMware networking • SSH • systemd troubleshooting • packet capture • TCP/UDP session inspection • PCAP handling • OpenSearch setup • service authentication • SOC network-analysis infrastructure

## Scope and Limitations

This is a **single-sensor VMware NAT lab**. Arkime monitors traffic visible to the guest's `ens33` interface; this does **not** imply full visibility into all host or LAN traffic. No malicious activity was established. There was no production deployment, high-availability testing, or large-scale packet ingest.

## Repository Contents

```text
.
├── README.md
├── .gitignore
├── architecture/
│   └── README.md
├── docs/
│   └── troubleshooting.md
└── screenshots/
    └── README.md
```

## Security Notes

Never publish OpenSearch administrator passwords, Arkime user credentials, S2S secrets, session cookies, local environment files, or unreviewed packet captures. Lab IP addresses and screenshots should be checked for private information before publishing.

## Future Work

- Add sanitized evidence screenshots.
- Import a small, authorized sample PCAP and demonstrate a protocol-focused search.
- Revisit each component for deeper SOC interview preparation and architecture explanation.
