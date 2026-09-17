# 🏢 Home-Lab Infrastructure & Network Architecture

A documentation of my dedicated home-lab environment designed for practical implementation of **network segmentation (VLANs)**, **firewall policies**, **virtualization**, and **homelab services**.

---

## 🗺️ Network Topology

![Network Topology](./topology.png)

---

## ⚙️ Hardware Infrastructure

| Host / Node | Hardware Specs | OS / Hypervisor | Role / Workloads |
| :--- | :--- | :--- | :--- |
| **Primary Node** | Lenovo ThinkCentre M720q (Intel Core i5-8500T, 6C/6T) | **Proxmox VE 9.2** | Virtualization Host, OPNsense Firewall/Router (VM 111), LXC Containers & Docker VMs |
| **Edge & Uplink** | AVM FRITZ!Box (`192.168.178.1`) | FRITZ!OS | WAN Gateway, Upstream Internet Router |
| **Storage & Edge Node** | Raspberry Pi 5 (`192.168.178.149`) | **Linux** | Lightweight Edge Services, File Storage & Network Shares |
| **Virtual Switching** | Linux Bridge (`vmbr0`) & Virtual Interfaces | Proxmox / OPNsense | IEEE 802.1Q VLAN Tagging (`vtnet0`–`vtnet4`) & Inter-VLAN Routing |

---

## 🛡️ Network Segmentation & Security Zones

The internal network is segmented into isolated broadcast domains using strict stateful firewall policies and RFC 1918 filtering:

| Interface / VLAN | Subnet | Gateway | DHCP Range | Zone Name | Description & Security Policy |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **WAN** (`vtnet0`) | `192.168.178.0/24` | `192.168.178.1` | Static (`.140`) | **WAN Uplink** | Upstream link to FRITZ!Box. Dedicated HTTPS rule configured for secure Web-GUI administration. |
| **LAN / VLAN 10** (`vtnet1`) | `10.0.10.0/24` | `10.0.10.1` | `.10` – `.245` | **Trusted LAN** | Admin workstations and trusted clients. Full access to WAN and unilateral routing to isolated segments. |
| **VLAN 20** (`vtnet2`) | `10.0.20.0/24` | `10.0.20.1` | `.100` – `.200` | **DMZ** | Public-facing services & reverse proxies (Nginx Proxy Manager). Isolated from internal networks. |
| **VLAN 30** (`vtnet3`) | `10.0.30.0/24` | `10.0.30.1` | `.100` – `.200` | **IoT** | Smart-home infrastructure (Home Assistant, bridges, IoT endpoints). Internet access allowed; RFC 1918 denied. |
| **VLAN 40** (`vtnet4`) | `10.0.40.0/24` | `10.0.40.1` | `.100` – `.200` | **Sandbox** | Ephemeral testing environments, malware/script analysis, and lab experiments. Zero lateral movement. |

---

## 🔒 Security Implementations

* **RFC 1918 Isolation (`Private_Netze`):** Central firewall alias defined across `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.
* **Inverted Egress Filtering (`!Private_Netze`):** Isolated segments (DMZ, IoT, Sandbox) can route outbound traffic exclusively to public WAN destinations while dropping all inter-segment and FRITZ!Box-bound lateral packets.
* **Inter-VLAN Service Pinhole:** Granular firewall rule on VLAN 20 allowing DMZ Ingress (`10.0.20.100`) to initiate stateful TCP sessions strictly to Jellyfin (`10.0.10.101:8096`) in VLAN 10, maintaining full isolation from all other LAN endpoints.
* **Controlled Core Services:** Explicit rule allowing TCP/UDP port 53 (DNS) directly to `This Firewall` (`OPNsense`) while dropping unauthorized internal traffic.
* **Stateful Inter-VLAN Administration:** Trusted LAN initiates stateful sessions into DMZ, IoT, and Sandbox; return packets are allowed dynamically while unsolicited connections back to LAN remain strictly blocked.
* **Upstream Static Routing:** Configured static IPv4 route on FRITZ!Box (`10.0.0.0/16` via `192.168.178.140`) to enable seamless two-way routing for upstream clients to downstream VLANs without double NAT.
* **Verified Containment:** Validated via isolated container testing (LXC `120 (test-web)`) confirming zero packet loss to WAN DNS (`google.com`) and 100% packet drop against host gateways (`192.168.178.1`).

---

## 📌 Roadmap & Future Enhancements

- [x] Hypervisor-level VLAN tagging and network bridge configuration
- [x] OPNsense multi-interface routing and isolated DHCP pools
- [x] RFC 1918 firewall isolation matrix and container verification
- [x] Migration of existing workloads (Nginx Proxy Manager to DMZ, Home Assistant to IoT, Jellyfin to LAN)
- [ ] Centralized DNS sinkhole via AdGuard Home
- [ ] Centralized log aggregation and telemetry (Grafana / Prometheus / Syslog)
- [ ] Implementation of a SIEM solution for intrusion detection (Wazuh / CrowdSec)
