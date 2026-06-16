# 🛡️ Network Security Monitoring (NSM) Lab

A 3-layer Network Security Monitoring stack built on a virtualized internal network (`192.168.56.0/24`), combining raw packet capture, protocol intelligence, and active threat detection.

---

## 🧪 Lab Architecture

```
┌─────────────────────────────────────────────┐
│         Host-Only Network: 192.168.56.0/24  │
│                                             │
│  ┌──────────────┐     ┌──────────────────┐  │
│  │ Ubuntu Desktop│     │   Windows 10     │  │
│  │  (Attacker)  │     │   (Target)       │  │
│  └──────┬───────┘     └────────┬─────────┘  │
│         │                      │             │
│         └──────────┬───────────┘             │
│                    │                         │
│         ┌──────────▼──────────┐              │
│         │  Ubuntu Server      │              │
│         │  (Monitor - enp0s8) │              │
│         │  Promiscuous Mode   │              │
│         │  ┌───────────────┐  │              │
│         │  │    TShark     │  │ Layer 1      │
│         │  │    Zeek       │  │ Layer 2      │
│         │  │    Suricata   │  │ Layer 3      │
│         │  └───────────────┘  │              │
│         └─────────────────────┘              │
└─────────────────────────────────────────────┘
```

The monitoring server's `enp0s8` interface is set to **Promiscuous Mode (Allow VMs)** to capture all lateral traffic between endpoints.

---

## 🔧 Tools & Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| Layer 1 | **TShark** | Raw packet capture & forensic PCAP recording |
| Layer 2 | **Zeek** | Protocol intelligence — structured connection logs |
| Layer 3 | **Suricata** | IDS — signature-based real-time threat detection |

---

## 📋 Command Reference

### System Setup

```bash
# Connect to monitoring server via SSH
ssh username@<server_IP>

# Update system repositories
sudo apt update
```

### Layer 1 — TShark (Raw Packet Capture)

```bash
# Capture live traffic on the host-only interface
sudo tshark -i enp0s8

# Capture only ICMP (ping) traffic
sudo tshark -i enp0s8 -f "icmp"

# Record traffic to file for 40 seconds
sudo tshark -i enp0s8 -w /tmp/capture.pcapng -a duration:40

# Read a saved capture file
sudo tshark -r /tmp/capture.pcapng

# Filter for ICMP traffic in a recorded file
sudo tshark -r /tmp/capture.pcapng -Y icmp
```

### Layer 2 — Zeek (Protocol Intelligence)

```bash
# Feed a capture file to Zeek for offline analysis
cd /tmp
sudo /opt/zeek/bin/zeek -r capture.pcapng

# View the structured connection log
cat con.log

# Deploy Zeek in live monitoring mode
sudo /opt/zeek/bin/zeekctl deploy

# Check Zeek operational status
sudo /opt/zeek/bin/zeekctl status

# Monitor live connection data in real time
sudo tail -f /opt/zeek/spool/zeek/con.log
```

### Layer 3 — Suricata (Active Threat Detection)

```bash
# Install Suricata
sudo apt install suricata

# Enable auto-start on boot
sudo systemctl enable suricata

# Pull latest Emerging Threats ruleset (60,000+ signatures)
sudo suricata-update

# Validate config and custom rules for syntax errors
sudo suricata -t -c /etc/suricata/suricata.yaml

# Restart Suricata to apply changes
sudo systemctl restart suricata

# Watch alerts in real time
sudo tail -f /var/log/suricata/fast.log

# Clear alert log for fresh testing
sudo truncate -s 0 /var/log/suricata/fast.log
```

### Attacker Simulation (Run from Ubuntu Desktop)

```bash
# Generate HTTP traffic to trigger User-Agent detection
sudo apt install curl
curl http://<server_IP>

# Launch a stealthy TCP SYN port scan against Windows VM
sudo apt install nmap
sudo nmap -sS <windows_IP>
```

---

## ✍️ Custom Suricata Rules

Located in `/var/lib/suricata/rules/custom.rules`

### Rule 1 — Suspicious User-Agent Detection

```
alert http any any -> any any (
  msg:"Suspicious non-browser HTTP client detected";
  content:"curl";
  http_user_agent;
  sid:1000001;
  rev:1;
)
```

**Logic:** Inspects outbound HTTP `User-Agent` headers. Fires when `curl` is detected, indicating command-line tooling rather than a legitimate browser — a common early indicator of scripted reconnaissance or data exfiltration.

---

### Rule 2 — Internal Port Scan Detection

```
alert tcp $HOME_NET any -> $HOME_NET any (
  msg:"Internal Port Scan Detected (SYN Flood)";
  flags:S;
  threshold:type threshold, track by_src, count 50, seconds 30;
  sid:1000002;
  rev:1;
)
```

**Logic:** Tracks all internal-to-internal TCP SYN packets. If a single host sends 50+ SYN requests within 30 seconds, it flags an active port sweep — a core technique in lateral movement and pre-ransomware reconnaissance.

---

### Rule 3 — SMB Enumeration Probe

```
alert tcp $HOME_NET any -> $HOME_NET 445 (
  msg:"Potential Ransomware SMB Enumeration Probe";
  flags:S;
  sid:1000003;
  rev:1;
)
```

**Logic:** Triggers on any internal TCP connection attempt to Port 445 (SMB). A critical indicator of self-propagating ransomware (e.g. WannaCry) scanning for vulnerable internal targets.

---

## 📁 Repository Structure

```
nsm-lab/
├── README.md               # This file
├── rules/
│   └── custom.rules        # All custom Suricata detection rules
├── docs/
│   └── setup-notes.md      # Extended lab setup documentation
├── captures/
│   └── .gitkeep            # PCAP samples (add your own)
└── screenshots/
    └── .gitkeep            # Evidence screenshots from detection tests
```

---

## 🎯 Skills Demonstrated

- Virtualbox host-only network configuration with promiscuous mode
- Packet capture and forensic analysis with TShark/Wireshark
- Protocol log analysis with Zeek (`conn.log`, `dns.log`, `http.log`)
- IDS deployment and rule tuning with Suricata + Emerging Threats ruleset
- Custom Suricata rule authoring (threshold, content matching, flag detection)
- Attacker simulation: curl-based HTTP probes, Nmap SYN scans
- Real-time alert monitoring with `fast.log`

---

## 📌 References

- [Suricata Documentation](https://suricata.readthedocs.io/)
- [Zeek Documentation](https://docs.zeek.org/)
- [Emerging Threats Open Ruleset](https://rules.emergingthreats.net/)
- [TryHackMe SOC Level 1 Path](https://tryhackme.com/path/outline/soclevel1)
