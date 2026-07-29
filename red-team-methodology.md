---
description: A concise host-discovery reference for authorized red-team assessments.
---

# Red Team Methodology

> Use these commands only on systems and networks you are explicitly authorized to assess.

## Host Discovery

### Local-network discovery

<details>
<summary>ARP sweep <sup>· PJPT</sup></summary>

```bash
sudo arp-scan --localnet
```

- Finds active IPv4 devices on the directly connected Layer 2 network.
- Records IP and MAC addresses for reachable hosts.
</details>

<details>
<summary>Active ARP discovery <sup>· PJPT</sup></summary>

```bash
sudo netdiscover -r 192.168.1.0/24
```

- Sends ARP requests across the specified local subnet.
- Use when the target range is on the current network segment.
</details>

<details>
<summary>ICMP range sweep <sup>· PJPT</sup></summary>

```bash
fping -a -g 10.10.10.0/24 2>/dev/null
```

- Lists addresses that answer ICMP echo requests.
- A missing reply does not prove that a host is offline.
</details>

### Routed-network discovery

<details>
<summary>TCP SYN and ACK probes <sup>· CPTS</sup></summary>

```bash
sudo nmap -sn -PS22,80,443 -PA80,443 10.10.10.0/24
```

- Identifies hosts that respond to selected TCP SYN or ACK probes.
- Useful when ICMP echo requests are filtered.
</details>

<details>
<summary>UDP service probes <sup>· CPTS</sup></summary>

```bash
sudo nmap -sn -PU53,123,161 10.10.10.0/24
```

- Uses UDP probes to elicit responses from common infrastructure services.
- Helps identify hosts that do not answer ICMP or TCP discovery probes.
</details>

<details>
<summary>Nmap host-only sweep <sup>· CPTS</sup></summary>

```bash
nmap -sn -n 10.10.10.0/24
```

- Performs Nmap host discovery without a port scan.
- Disables reverse DNS lookups for cleaner, faster output.
</details>
