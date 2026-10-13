# Architecture — Arkime Packet Indexing Lab

## Data Flow

```text
Traffic visible to Ubuntu ens33 (VMware NAT)
                      |
                      v
               Arkime Capture
                  /       \
                 v         v
       Raw PCAP files   Session metadata
       /opt/arkime/raw        |
                              v
                      OpenSearch 3.9.0
                              |
                              v
                        Arkime Viewer
                        TCP port 8005
                              |
                              v
                 Windows browser / analyst
```

**Arkime Capture** records traffic and extracts session metadata. **OpenSearch** stores searchable metadata, while **PCAP files** retain the underlying packet evidence. **Arkime Viewer** provides a browser interface to search sessions and export original packets.

## Platform

- Dedicated VMware Workstation VM, Ubuntu Server 24.04 LTS
- NAT-connected `ens33` interface
- 2 vCPUs, 4 GB RAM, 30 GB disk
- Arkime 6.3.1, OpenSearch 3.9.0
- OpenSearch configured with `-Xms1g` and `-Xmx1g`
- OpenSearch API on `https://localhost:9200` with authentication and local self-signed TLS
- Arkime web interface on port 8005

## Security and Visibility Boundaries

- The host can access the Arkime web interface over the VMware NAT network.
- The capture sensor can only monitor traffic visible on `ens33`. NAT mode does **not** automatically mirror every Windows host or LAN packet into this VM.
- Local self-signed TLS was used for the OpenSearch API in the test lab; Arkime's database utility required `--insecure` for certificate verification during the exercised commands. This is **not** a production TLS recommendation.
- Web access used HTTP inside the lab. Production deployments should use proper HTTPS and access controls.
- OUI enrichment provides MAC manufacturer names; MaxMind GeoIP/ASN enrichment was intentionally skipped.

## What Was Verified

Four sessions were observed in Arkime Viewer, including SSDP, mDNS and TCP traffic. One TCP session on port 8005 was expanded and its PCAP downloaded. The prior `example.com` test request was not individually verified in those indexed results.
