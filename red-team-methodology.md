# Red Team Methodology

## Method Map

- `1` **Host Discovery**
  - `1.1` **Local-network discovery**
    - `1.1.1` ARP host discovery
    - `1.1.2` Netdiscover ARP discovery
    - `1.1.3` ICMP range sweep
  - `1.2` **Routed-network discovery**
    - `1.2.1` TCP SYN and ACK probes
    - `1.2.2` UDP service probes
    - `1.2.3` Nmap host-only sweep

## 1. Host Discovery

### 1.1 Local-network discovery

#### 1.1.1 ARP host discovery <sup>· PJPT</sup>

```bash
sudo arp-scan -I <INTERFACE> --localnet
```

<details>
<summary>Details</summary>

**Description**

- Sends ARP requests to find active IPv4 hosts on the selected local Layer 2 network.
- Records IP, MAC, and vendor information for responding hosts.

**Parameters**

- `-I <INTERFACE>` — Selects the network interface to use.
- `--localnet` — Derives the target range from that interface’s IP address and netmask.

</details>

#### 1.1.2 Netdiscover ARP discovery <sup>· PJPT</sup>

```bash
sudo netdiscover -i <INTERFACE> -r <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Sends ARP requests across a specified local subnet.
- Can also be used for passive ARP reconnaissance.

**Parameters**

- `-i <INTERFACE>` — Selects the interface for ARP traffic.
- `-r <IP/MASK>` — Sets the local CIDR range to scan.

</details>

#### 1.1.3 ICMP range sweep <sup>· CPTS</sup>

```bash
fping -a -g <IP/MASK> 2>/dev/null
```

<details>
<summary>Details</summary>

**Description**

- Lists addresses that answer ICMP echo requests.
- A missing reply does not prove that a host is offline.

**Parameters**

- `-a` — Shows only hosts that respond.
- `-g` — Generates addresses from the supplied CIDR range.
- `<IP/MASK>` — Target CIDR range.
- `2>/dev/null` — Suppresses error output from the shell.

</details>

### 1.2 Routed-network discovery

#### 1.2.1 TCP SYN and ACK probes <sup>· CPTS</sup>

```bash
sudo nmap -sn -PS22,80,443 -PA80,443 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Identifies hosts that respond to selected TCP SYN or ACK probes.
- Useful when ICMP echo requests are filtered.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PS22,80,443` — Sends TCP SYN probes to the listed ports.
- `-PA80,443` — Sends TCP ACK probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

#### 1.2.2 UDP service probes <sup>· CPTS</sup>

```bash
sudo nmap -sn -PU53,123,161 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Uses UDP probes to elicit responses from common infrastructure services.
- Helps identify hosts that do not answer ICMP or TCP discovery probes.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PU53,123,161` — Sends UDP probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

#### 1.2.3 Nmap host-only sweep <sup>· CPTS</sup>

```bash
nmap -sn -n <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Performs Nmap host discovery without a port scan.
- Disables reverse DNS lookups for cleaner, faster output.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-n` — Disables reverse DNS resolution.
- `<IP/MASK>` — Target host or CIDR range.

</details>
