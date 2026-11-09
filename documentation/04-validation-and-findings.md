# 04 — Validation and Findings

What was tested and what was actually observed. Only findings supported by the evidence are reported.

## Validation Summary

| Area | What was checked | Result / observation |
|---|---|---|
| VM connectivity | Attacker (LAN) reaching victim (OPT1) | Packets from `192.168.1.140` were captured on the victim `192.168.2.10` |
| Network segmentation | Two separate subnets joined only by OPNsense | `192.168.1.0/24` and `192.168.2.0/24`; attacker and victim are in different subnets |
| OPNsense | Interfaces and traffic between segments | Three interfaces configured; the attack traffic was not blocked |
| Suricata | Alerts during the attacks | 10 alerts from `192.168.1.140` to `192.168.2.10` on OPT1: port-scan and SSH brute-force signatures |
| Wireshark | Packets from the attacker | 2,056 SYN-only packets from the attacker out of 8,914 captured |
| Nmap / Hydra | Attacks against the lab victim | Detected by Suricata and visible in Wireshark (see findings) |

## Finding 01 — Port scan (Nmap) detected by Suricata and confirmed in Wireshark

- **Date:** 2026-10-14 (alert timestamps)
- **Objective:** Check whether Suricata detects a port scan against the victim.
- **Environment:** Kali `192.168.1.140` → Debian `192.168.2.10`, monitored on OPT1.
- **Tools:** Nmap, Suricata, Wireshark.

**What Suricata showed.** Five alerts from the same source IP and the same source port (`53105`), each to a different destination port:

| SID | Signature | Destination port | Service |
|---|---|---|---|
| 2010935 | ET SCAN Suspicious inbound to MSSQL port 1433 | 1433 | Microsoft SQL Server |
| 2010936 | ET SCAN Suspicious inbound to Oracle SQL port 1521 | 1521 | Oracle |
| 2010937 | ET SCAN Suspicious inbound to mySQL port 3306 | 3306 | MySQL |
| 2010939 | ET SCAN Suspicious inbound to PostgreSQL port 5432 | 5432 | PostgreSQL |
| 2002910 | ET SCAN Potential VNC Scan 5800-5820 | 5801 | VNC |

<!-- Signature names are truncated in the screenshot. Confirm each full name with the Info (pencil) button in the Alerts tab. -->

**What Wireshark showed.** Filtering the attacker's initial SYN packets returned 2,056 of 8,914 captured packets (23.1%). The packets share the same pattern: 60 bytes, `Len=0`, `Win=1024`, `MSS=1460`, one SYN per destination port (for example 443, 5900, 113, 995, 135, 587, 1723, 8888, 110, 3389), roughly one millisecond apart.

**Why this is a scan.** One source touching many different service ports on one host in a very short time is not normal client behavior. A normal client opens a few connections to the ports it needs. Each signature above fires because a different service port was probed. The small identical SYN packets are consistent with Nmap's default SYN scan.

**Triage conclusion.**
- **Verdict:** True positive (expected, I ran the scan). Reconnaissance activity.
- **Severity:** Low to medium. A scan alone does not mean compromise, but it shows an attacker mapping open services.
- **In a real SOC:** check whether the same source continues with more targeted activity (see Finding 02), then block or escalate according to policy.
- **Evidence:** [suricata-alerts.png](../screenshots/suricata-alerts.png), [wireshark-02-tcp-flags-analysis.png](../screenshots/wireshark-02-tcp-flags-analysis.png)

## Finding 02 — SSH brute force (Hydra) detected by Suricata

- **Date:** 2026-10-17, 16:59 (UTC−6)
- **Objective:** Check whether Suricata detects an SSH brute-force attempt.
- **Environment:** Kali `192.168.1.140` → Debian `192.168.2.10`, port 22, monitored on OPT1.
- **Tools:** Hydra, Suricata.

**What Suricata showed.** Three alerts, all to destination port 22, each with a different source port:

| SID | Signature | Source port |
|---|---|---|
| 2006546 | ET SCAN LibSSH Based Frequent SSH Connections Likely BruteForce Attack | 57810 |
| 2006546 | (same signature) | 54232 |
| 2001219 | ET SCAN Potential SSH Scan | 54228 |

<!-- The 2001219 name is truncated in the screenshot. Confirm it with the Info button. -->

Alert detail (first row): protocol TCP, interface OPT1, configured action `Alert`.

**Why this is a brute force.** A brute-force tool opens many SSH connections to the same service in a short time, and each connection uses a new source port. Several different source ports hitting port 22 from one IP is that pattern. SID 2006546 looks for frequent SSH connections from clients built on libssh, which Hydra's SSH module uses.

**What this does not prove.** The alert shows that the attempts happened, not that a password was guessed. To know the result, check the victim's SSH authentication log for successful logins from `192.168.1.140`.

TODO: add the outcome of the Hydra run (was a valid login found or not?).

**Triage conclusion.**
- **Verdict:** True positive (expected, I ran the attack). Credential attack against SSH.
- **Severity:** Higher than the scan, because it targets authentication directly.
- **In a real SOC:** review the target's auth logs for any successful login, block the source, and consider limiting SSH exposure, key-based login or rate limiting.
- **Evidence:** [suricata-alerts.png](../screenshots/suricata-alerts.png), [Alert-Analysis.png](../screenshots/Alert-Analysis.png)

## Combined observation

The scan alerts (2026-10-14) and the SSH alerts (2026-10-17) come from the same source IP against the same victim, and in all of them the action is `allowed`. The Wireshark capture on the victim confirms the attacker's packets arrived, so in this lab the traffic was detected but not blocked. This is expected: Suricata runs in IDS mode.

## Limitations

- This is a small personal virtual lab, not a production network.
- Suricata inspects only the OPT1 interface.
- An IDS alert means a rule matched observed traffic. It does not by itself prove a real compromise.
- Firewall rules between LAN and OPT1 were not documented in this project.
- All tests were limited to machines I own.
