# Lab Setup Notes

## VirtualBox Network Configuration

- Network type: **Host-Only Adapter**
- Subnet: `192.168.56.0/24`
- Monitoring interface: `enp0s8`
- Promiscuous mode: **Allow VMs** (required to capture inter-VM traffic)

## VMs

| VM | Role | OS |
|----|------|----|
| Ubuntu Server | Monitor (TShark + Zeek + Suricata) | Ubuntu Server 22.04 |
| Ubuntu Desktop | Attacker / Endpoint | Ubuntu Desktop |
| Windows 10 | Target endpoint | Windows 10 |

## Zeek Installation Path

```
/opt/zeek/bin/zeek
/opt/zeek/bin/zeekctl
/opt/zeek/spool/zeek/conn.log
```

## Suricata Config & Logs

```
/etc/suricata/suricata.yaml       # Main config
/var/lib/suricata/rules/          # Rule directory
/var/log/suricata/fast.log        # Alert log
```

## Key Tips

- Always run `sudo suricata-update` before testing to pull the latest ET rules
- Use `tmux` with split panes to monitor `fast.log` and `conn.log` simultaneously
- Clear `fast.log` with `truncate -s 0` between test runs to avoid noise
- Verify Zeek is running with `zeekctl status` before expecting `conn.log` output
