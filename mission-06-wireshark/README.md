# Mission 06 — Wireshark Packet Capture & Traffic Analysis

## Objective

Install Wireshark/tshark, capture live network traffic on the Enterprise-D bridge system, and analyze it for suspicious packets and TCP/IP conversations.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2)
- **Tools:** Wireshark, tshark (CLI capture/analysis)

## Steps Performed

### 1. Installed Wireshark and tshark

```bash
echo "wireshark-common wireshark-common/install-setuid boolean true" | sudo debconf-set-selections
sudo DEBIAN_FRONTEND=noninteractive apt install wireshark tshark -y
```

Preseeding the `debconf` answer for the non-root capture prompt avoided an interactive dialog hang during unattended install.

### 2. Configured non-root packet capture

```bash
sudo usermod -aG wireshark $USER
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
```

### 3. Captured live traffic

Since WSL2 exposes only its virtual `eth0` interface (no direct access to physical adapters), traffic was captured on `eth0` via the CLI tool (`tshark`) rather than the GUI, which is more reliable in a headless WSL environment:

```bash
sudo tshark -i eth0 -w /tmp/enterprise-d-capture.pcap
```

Generated real traffic during capture (`curl`, `ping`) to have meaningful packets to analyze.

### 4. Analyzed the capture

```bash
# TCP conversation summary
tshark -r /tmp/enterprise-d-capture.pcap -z conv,tcp -q

# Filter by destination IP
tshark -r /tmp/enterprise-d-capture.pcap -Y "ip.addr==8.8.8.8"

# HTTP traffic only
tshark -r /tmp/enterprise-d-capture.pcap -Y "http"
```

Successfully identified and filtered ICMP echo request/reply pairs and other TCP/IP conversations from the capture.

### 5. Exported the capture for GUI inspection

Copied the `.pcap` file to the Windows host for deeper visual inspection in the desktop Wireshark GUI:

```bash
cp /tmp/enterprise-d-capture.pcap /mnt/c/Users/.../OneDrive/Desktop
```

## Result

Full packet capture and analysis pipeline validated: traffic captured on the lab system, filtered by IP/protocol via `tshark`, and exported for visual inspection — demonstrating the traffic-analysis skill required to spot anomalous or malicious network activity.

## Screenshots

See [`/docs/screenshots/mission-06/`](../docs/screenshots/mission-06/) for the captured traffic.
