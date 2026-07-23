# Linux Network Administration



## Checking Your Network from the Command Line
When working on servers or troubleshooting without a GUI, command-line tools are mandatory.

### The `ip` Command Suite
The legacy `net-tools` (like `ifconfig`) have been replaced by the `iproute2` package.

* **View IP Addresses:** `ip addr show` (or simply `ip a`) displays the IP addresses assigned to all network interfaces.
* **View Link Status:** `ip link show` displays the physical state (`UP`/`DOWN`) and MAC addresses of network devices.
* **Interface Statistics:** `ip -s link show` displays packet transmission statistics, which is useful for identifying dropped packets or network errors.

---

## Checking Connectivity to Remote Systems
When a network connection fails, administrators use a standard sequence of tools to isolate the problem.

* **`ping`:** Uses ICMP Echo Request packets to verify host reachability. It is the first step in determining if a remote machine is online.
  * *Example:* `ping -c 4 8.8.8.8` (sends 4 packets and stops).
* **`traceroute` / `tracepath`:** Tracks the path packets take across the network to reach a destination. It helps identify exactly which router or hop is dropping the connection.
* **`ss` (Socket Statistics):** Replaces the older `netstat` command. It dumps socket statistics to help you see what ports are listening for connections and what active connections exist.
  * *Example:* `ss -tulpn` shows all active listening TCP/UDP ports and the associated processes.
* **`telnet` / `nc` (Netcat):** Useful for testing if a specific TCP port is open on a remote machine.
  * *Example:* `nc -vz 192.168.1.50 80` tests if port 80 is reachable.

---

## 4. Checking Routing Information
Your system needs a routing table to know where to send network traffic.

* **Viewing the Route Table:** The command `ip route show` displays the current routing table.
* **Understanding the Output:**
  * `default via <gateway_ip>`: Indicates the default gateway. Any traffic not destined for a local subnet is sent to this router.
  * **Directly Connected Routes:** Show local subnets attached directly to your network interfaces (e.g., `192.168.1.0/24 dev eth0`).
* **Adding/Deleting Routes:** Administrators can manipulate routes temporarily using `ip route add` or permanently by configuring connection profiles.

---

## System Configuration Files

Even with dynamic management tools, several core text files fundamentally govern how Linux handles naming and network resolution.

| Configuration File | Primary Function |
| :--- | :--- |
| **/etc/hosts** | A static lookup table for hostnames to IP addresses. The system checks this file before querying a DNS server. It is highly useful for local, small-scale network configurations or overriding DNS. |
| **/etc/resolv.conf** | Defines the DNS (Domain Name System) servers your system queries to resolve domain names (like google.com) into IP addresses. NetworkManager typically auto-generates this file based on DHCP leases. |
| **/etc/nsswitch.conf** | The Name Service Switch file. It dictates the priority order in which the system searches for information. For networking, the `hosts:` line usually reads `files dns`, meaning the system checks `/etc/hosts` (files) first, then falls back to `/etc/resolv.conf` (dns). |

---

## Configuring Linux as a Router (IP Forwarding)

By default, a Linux system will drop network packets that are not explicitly destined for its own IP addresses. To act as a router or VPN gateway, IP forwarding must be enabled.

### Temporary Enablement
You can turn on IPv4 forwarding instantly by modifying the kernel parameter in the `/proc` filesystem:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### Permanent Enablement
To ensure the setting survives a reboot, you must configure `sysctl`. Add or modify the following line in `/etc/sysctl.conf` (or a dedicated file in `/etc/sysctl.d/`):

```text
net.ipv4.ip_forward = 1
```

```bash
sysctl -p
```

### Firewall Rules
Simply enabling IP forwarding is not enough. You must also configure `firewalld` or `iptables` to permit traffic forwarding between interfaces and, if acting as a gateway to the internet, set up NAT (Network Address Translation) or masquerading.