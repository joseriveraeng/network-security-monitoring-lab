# 01 — Lab Overview

## Purpose

This project is a small network security monitoring lab built to practice the daily workflow of a SOC Analyst L1: **detect, investigate and document suspicious network activity**.

An attacker machine (Kali Linux) runs a port scan (Nmap) and an SSH brute-force attack (Hydra) against a victim machine (Debian). All traffic between them crosses an OPNsense firewall, where Suricata (IDS) generates alerts. The same traffic is then inspected packet by packet in Wireshark on the victim.

## Learning Objectives

- Understand how traffic moves between two segmented networks through a firewall.
- Configure Suricata as an IDS on an OPNsense interface.
- Read and interpret Suricata alerts (who attacked whom, with what signature, and how often).
- Confirm an alert by finding the matching packets in Wireshark.
- Document each finding with evidence and a triage conclusion.

## Components

| Component | Function in this lab |
|---|---|
| Kali Linux | Attacker: runs Nmap (port scan) and Hydra (SSH brute force) |
| Debian | Victim: receives the attacks; Wireshark captures its traffic |
| OPNsense | Firewall/router between the networks; hosts Suricata |
| Suricata | IDS: matches traffic against rules and raises alerts |
| Wireshark | Packet-level inspection on the victim |
| Nmap | Port scanning (reconnaissance) |
| Hydra | SSH login brute-force tool |
| Oracle VirtualBox | Virtualization platform |

## Scenario

| Role | Machine | IP address | Network |
|---|---|---|---|
| Attacker | Kali Linux | `192.168.1.140` | LAN (`192.168.1.0/24`) |
| Victim | Debian | `192.168.2.10` | OPT1 (`192.168.2.0/24`) |
| Firewall / IDS | OPNsense | `192.168.1.1` / `192.168.2.1` | Between both networks |

## Scope

All activity was performed in a personal virtual lab against machines I own. This documentation does not claim production deployment, enterprise-scale monitoring or a complete SOC environment.
