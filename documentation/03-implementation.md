# 03 — Implementation Notes

How the lab was built and how the alerts and packets were reviewed. No passwords or credentials are published.

## 1. Virtual Machine Setup

Three VMs were created in Oracle VirtualBox: `kali-linux-2026.2-virtualbox-amd64`, `OPNsense` and `debian`.

Evidence: [Virtual_Machines.png](../screenshots/Virtual_Machines.png)

## 2. OPNsense Network Configuration

OPNsense (version 26.7) was configured with three interfaces. LAN and OPT1 were assigned to separate VirtualBox internal networks, so Kali and Debian live on different subnets and can only talk through the firewall. Addresses are listed in [02-network-architecture.md](02-network-architecture.md).

Evidence:
- Console menu with interface addresses: [OPNsense_networks.png](../screenshots/OPNsense_networks.png)
- Web dashboard (Interfaces widget): [opnsense-interfaces.png](../screenshots/opnsense-interfaces.png)

## 3. Suricata IDS

Configured in **Services → Intrusion Detection → Administration**.

| Setting | Value |
|---|---|
| Enabled | Yes |
| Capture mode | PCAP live mode (IDS) |
| Promiscuous mode | Enabled |
| Interfaces | OPT1 |
| Home networks | `192.168.2.0/24` |
| Pattern matcher / Detect profile | Default / Default |

- **IDS mode** means Suricata only raises alerts; it does not block traffic. That is why the alerts show the action `allowed`.
- **Home networks** tells Suricata which addresses are "inside". Many rules are written relative to it, so it must include the protected subnet (the victim's `192.168.2.0/24`).
- The **Rules** tab lists 1,164 rules (action `alert`, enabled).

Evidence:
- [suricata-configuration.png](../screenshots/suricata-configuration.png)
- [suricata-enabled-rules.png](../screenshots/suricata-enabled-rules.png)

## 4. Attack Generation (from Kali, against the lab victim only)

```bash
# Port and service-version scan
nmap -sV 192.168.2.10

# SSH brute force: one username, many passwords from a wordlist
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.2.10
```

- **Nmap `-sV`** scans the victim's ports and then probes the open services to identify their versions. Run as root, it finds ports with SYN probes first, which is the pattern seen in Wireshark.
- **Hydra `-l root`** sets a single username and **`-P`** sets a file with passwords to try. `rockyou.txt` is the default password list that ships with Kali Linux. Hydra's SSH module opens many connections to port 22, which is what Suricata flagged.

## 5. How I Reviewed the Alerts in OPNsense

1. Open **Services → Intrusion Detection → Administration → Alerts**.
2. Pick the date/log to review and sort by timestamp.
3. Read each row: *who* (source) → *whom* (destination and port), *what* (alert name and SID).
4. Click the pencil icon (**Info**) on one alert to see the full detail.
5. Group alerts by source IP and by time: many alerts from one source in a short time usually means one tool, not many events.

How to read an alert:

| Field | Meaning | Example from this lab |
|---|---|---|
| Timestamp | When the rule matched | `2026-10-17T16:59:00 -0600` |
| Alert / SID | Signature name and its unique rule ID | `ET SCAN LibSSH Based Frequent SSH Connections Likely BruteForce Attack` / `2006546` |
| Protocol | Transport protocol | TCP |
| Source IP : port | Who started the traffic | `192.168.1.140:57810` |
| Destination IP : port | Target host and service | `192.168.2.10:22` (SSH) |
| Interface | Where Suricata saw it | OPT1 |
| Action | What happened to the traffic | `allowed` (IDS mode = alert only) |

Evidence:
- Alert list: [suricata-alerts.png](../screenshots/suricata-alerts.png)
- Alert detail: [Alert-Analysis.png](../screenshots/Alert-Analysis.png)

## 6. How I Inspected the Packets in Wireshark

Wireshark ran on the victim VM (interface `enp0s3`). Two views were used:

- **Normal traffic overview** (no filter): a live capture with ordinary traffic such as DNS and QUIC. It is the baseline to compare against. Evidence: [wireshark-01-network-traffic-overview.png](../screenshots/wireshark-01-network-traffic-overview.png)
- **Attacker traffic only**, with this display filter:

```text
ip.src == 192.168.1.140 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

| Part of the filter | What it does |
|---|---|
| `ip.src == 192.168.1.140` | Only packets sent by the attacker |
| `tcp.flags.syn == 1` | Only packets with the SYN flag (start of a TCP connection) |
| `tcp.flags.ack == 0` | Only initial SYNs, not the replies |

Evidence: [wireshark-02-tcp-flags-analysis.png](../screenshots/wireshark-02-tcp-flags-analysis.png)
