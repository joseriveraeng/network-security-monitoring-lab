# 05 — Evidence Index

Screenshots are in the `screenshots/` folder; the diagram is in `diagrams/`.

| File | What it shows | What it demonstrates |
|---|---|---|
| [`diagrams/network-topology.png`](../diagrams/network-topology.png) | WAN, OPNsense with Suricata, LAN 1 and LAN 2 | Network design |
| [`Virtual_Machines.png`](../screenshots/Virtual_Machines.png) | VirtualBox with Kali, OPNsense and Debian | Lab composition |
| [`OPNsense_networks.png`](../screenshots/OPNsense_networks.png) | OPNsense console: WAN `10.0.2.15/24`, LAN `192.168.1.1/24`, OPT1 `192.168.2.1/24` | Interface addressing |
| [`opnsense-interfaces.png`](../screenshots/opnsense-interfaces.png) | OPNsense web dashboard with the three interfaces | Network configuration |
| [`suricata-configuration.png`](../screenshots/suricata-configuration.png) | Suricata settings: IDS mode, OPT1, home network | IDS configuration |
| [`suricata-enabled-rules.png`](../screenshots/suricata-enabled-rules.png) | Rules tab with 1,164 rules | Ruleset in use |
| [`suricata-alerts.png`](../screenshots/suricata-alerts.png) | Alerts tab: scan and SSH alerts from `192.168.1.140` | Alert review |
| [`Alert-Analysis.png`](../screenshots/Alert-Analysis.png) | Detail of SID 2006546 (SSH brute force) | Alert analysis |
| [`wireshark-01-network-traffic-overview.png`](../screenshots/wireshark-01-network-traffic-overview.png) | Live capture of normal traffic | Baseline traffic |
| [`wireshark-02-tcp-flags-analysis.png`](../screenshots/wireshark-02-tcp-flags-analysis.png) | SYN-only packets from the attacker to many ports | Port scan confirmed in packets |

## Final Evidence Checklist

- [x] Three virtual machines are shown.
- [x] OPNsense interface addresses match the architecture document.
- [x] Suricata screenshot shows the monitored interface (OPT1).
- [x] At least one real alert is documented (scan and SSH brute force).
- [x] Wireshark screenshots have an explanation in the findings.
- [ ] Terminal screenshot of the Nmap scan (optional but recommended).
- [ ] Terminal screenshot of the Hydra run (optional but recommended).
- [ ] Topology diagram labeled with the attacker (Kali) and the victim (Debian).
- [ ] Public IP addresses hidden in `wireshark-01-network-traffic-overview.png`.
