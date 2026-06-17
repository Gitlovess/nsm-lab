# Network Monitoring

**Quick lookup for commands used in video.**  
**👉 Watch the video first: [Lab 2](https://youtu.be/JSP-yOBRbrg)**

---

## SSH & Update
```bash
ssh defender@192.168.56.101
sudo apt update
```

---

## TSHARK
```bash
# Install
sudo apt install tshark

# Basic capture
sudo tshark -i enp0s8
sudo tshark -i enp0s8 -f "icmp"

# Save & read
sudo tshark -i enp0s8 -w /tmp/capture.pcapng -a duration:40
sudo tshark -r /tmp/capture.pcapng
sudo tshark -r /tmp/capture.pcapng -Y "icmp"
```

---

## ZEEK
```bash
# Add repository
echo "deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/ /" | sudo tee /etc/apt/sources.list.d/zeek.list
curl -fsSL https://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/zeek.gpg > /dev/null

# Install
sudo apt update
sudo apt install zeek

# Analyze capture
cd /tmp
sudo /opt/zeek/bin/zeek -r capture.pcapng
cat conn.log
```

### Live Monitoring
```bash
sudo nano /opt/zeek/etc/node.cfg
# Change: interface=enp0s8

sudo nano /opt/zeek/etc/networks.cfg
# Add: 192.168.56.0/24    lab network

sudo /opt/zeek/bin/zeekctl deploy
sudo /opt/zeek/bin/zeekctl status
sudo tail -f /opt/zeek/spool/zeek/conn.log
```

---

## SURICATA
```bash
# Install
sudo apt install suricata
sudo systemctl enable suricata

# Update rules
sudo suricata-update
```

### Configuration
```bash
sudo nano /etc/suricata/suricata.yaml
# Set: HOME_NET: "192.168.56.0/24"
# Set: interface: enp0s8
# Set: default-rule-path: /var/lib/suricata/rules

# Test & restart
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl restart suricata
```

### Monitor alerts
```bash
tmux
sudo tail -f /var/log/suricata/fast.log
```

---

### Create Custom Rules File
```bash
sudo nano /var/lib/suricata/rules/custom.rules
```

**Paste rules from:** [suricata-custom.rules](./suricata-custom.rules)

Save: `Ctrl+X`, then `Y`, then `Enter`
```bash
sudo nano /etc/suricata/suricata.yaml
# Add under rule-files: - custom.rules

sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl restart suricata
```

---

## TESTING
```bash
# From Ubuntu Desktop
ping -c 5 192.168.56.101
curl http://192.168.56.101
sudo apt install nmap
sudo nmap -sS 192.168.56.102
```

---

## IP ADDRESSES
- Server: 192.168.56.101
- Ubuntu Desktop: 192.168.56.102
- Windows: 192.168.56.103

## TMUX
- Split: `Ctrl+B` then `%`
- Close: `Ctrl+B` then `X` then `Y`
- Exit: `Ctrl+D`
