# Network Security Monitoring Lab

A personal network security monitoring lab built with three virtual machines using Oracle VirtualBox. The project explores network segmentation, firewall functionality, intrusion detection, packet analysis, and basic network discovery in a controlled virtual environment.

## Objectives

- Understand network traffic flow between segmented virtual networks.
- Explore firewall interfaces and traffic filtering with OPNsense.
- Configure and observe Suricata IDS alerts and logs.
- Inspect network packets with Wireshark.
- Use Nmap for controlled network discovery and testing.
- Document observations from security monitoring activities.

## Lab Environment

| Component | Role |
|---|---|
| Kali Linux | Security testing and traffic generation |
| Debian | Internal client machine |
| OPNsense | Firewall and routing between virtual networks |
| Suricata | Intrusion Detection System (IDS) |
| Wireshark | Packet capture and analysis |
| Nmap | Network discovery and testing |

## Network Architecture

The lab uses three virtual machines connected through VirtualBox virtual networks. OPNsense provides the firewall and routing point between the internal networks.

![Network topology](diagrams/network-topology.png)

See [Network Architecture](documentation/02-network-architecture.md) for the addressing plan and topology details.

## Implementation

The work included:

1. Creating and configuring the three virtual machines.
2. Configuring OPNsense network interfaces and firewall functionality.
3. Enabling Suricata monitoring and reviewing its alerts and logs.
4. Capturing and inspecting traffic with Wireshark.
5. Performing controlled network discovery with Nmap.

More detail is available in the [Implementation Notes](documentation/03-implementation.md).

## Validation and Findings

The lab was used to observe traffic between virtual machines, review Suricata events, explore firewall filtering behavior, and inspect packet captures.

Specific test observations and evidence should be recorded in [Validation and Findings](documentation/04-validation-and-findings.md). Only findings that were personally observed in the lab are reported.

## Screenshots and Evidence

Screenshots are organized in the `screenshots/` directory. The [Evidence Index](documentation/05-evidence-index.md) describes what each image demonstrates.

## Skills Demonstrated

- Network security monitoring
- Intrusion Detection Systems (IDS)
- Firewall fundamentals and configuration
- Network segmentation and traffic flow
- Linux virtual machines
- Packet capture and analysis
- Network discovery and troubleshooting
- Technical documentation

## Disclaimer

This project was created for educational purposes in a personal virtual lab. Testing was limited to systems and networks controlled by the author.
