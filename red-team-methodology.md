# Red Team Methodology

## Host Discovery

### Local-network discovery

#### ARP sweep <sup>· PJPT</sup>

```bash
sudo arp-scan --localnet
```

<details>
<summary>Details</summary>

**Description**

- Finds active IPv4 devices on the directly connected Layer 2 network.
- Records IP and MAC addresses for reachable hosts.

**Parameters**

- `--localnet` — Scans the locally connected network inferred from the active interface.
</details>

#### Active ARP discovery <sup>· PJPT</sup>

```bash
sudo netdiscover -r <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Sends ARP requests across the specified local subnet.
- Use when the target range is on the current network segment.

**Parameters**

- `-r` — Sets the target range.
- `<IP/MASK>` — CIDR range on the directly connected network.
</details>

#### ICMP range sweep <sup>· PJPT</sup>

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

### Routed-network discovery

#### TCP SYN and ACK probes <sup>· CPTS</sup>

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

#### UDP service probes <sup>· CPTS</sup>

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

#### Nmap host-only sweep <sup>· CPTS</sup>

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
