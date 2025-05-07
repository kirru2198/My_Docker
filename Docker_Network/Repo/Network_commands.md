Here's a comprehensive list of **networking-related commands** you can use on an **Ubuntu** (or any Linux-based) machine, grouped by functionality:

---

### 🔍 **Check IP Address & Interface Info**

* `ip a` or `ip addr` — Show all IP addresses.
* `ifconfig` *(deprecated, use `ip`)* — Show network interfaces.
* `ip link` — Show network interface link-layer details.
* `ip -br addr` — Brief interface status.

---

### 📡 **Network Interface Management**

* `ip link set <iface> up/down` — Bring interface up or down.
* `ifup <iface>` / `ifdown <iface>` — Legacy commands to manage interfaces.
* `nmcli device` — List devices using NetworkManager.
* `nmcli device connect <iface>` — Connect interface via NetworkManager.

---

### 🌐 **Routing**

* `ip r` or `ip route` — Show routing table.
* `route -n` — Show routing table (older tool).
* `ip route add <network> via <gateway>` — Add a route.
* `ip route del <network>` — Delete a route.

---

### 🧭 **Hostname & DNS**

* `hostname` — Show or set hostname.
* `hostnamectl` — Manage hostname in systemd-based distros.
* `cat /etc/hostname` — View static hostname.
* `cat /etc/hosts` — View static host-name to IP mappings.
* `cat /etc/resolv.conf` — View DNS servers used.
* `nmcli dev show | grep DNS` — Show DNS servers via NetworkManager.

---

### 🔎 **Network Diagnostics**

* `ping <host>` — Check connectivity.
* `traceroute <host>` — Show packet route.
* `tracepath <host>` — Trace network path (alternative to traceroute).
* `mtr <host>` — Real-time traceroute + ping.
* `dig <domain>` — DNS lookup.
* `nslookup <domain>` — DNS query (legacy).
* `whois <domain>` — Get domain registration info.
* `curl <url>` — Fetch from HTTP/HTTPS endpoints.
* `wget <url>` — Download from web.

---

### 🛠️ **TCP/UDP & Sockets**

* `netstat -tulnp` *(deprecated)* — List open ports & listening services.
* `ss -tulnp` — Modern replacement for netstat.
* `lsof -i` — List open network files/ports.
* `nc -l -p <port>` — Netcat, listen on a port.
* `nc <host> <port>` — Connect to a remote host.
* `telnet <host> <port>` — Test connection to a port (legacy).
* `nmap <host>` — Scan open ports (requires installation).

---

### 🌐 **Firewall & Security**

* `ufw status` — Show firewall status.
* `ufw enable` / `ufw disable` — Enable/disable firewall.
* `iptables -L` — List rules (if using iptables).
* `nft list ruleset` — Show rules in `nftables` (modern firewall system).

---

### 📶 **Wi-Fi & Wireless Tools**

* `iwconfig` — Wireless interface info (deprecated).
* `iw dev` — New wireless tool.
* `nmcli dev wifi` — List available Wi-Fi networks.
* `nmcli dev wifi connect "<SSID>" password "<pass>"` — Connect to Wi-Fi.

---

### 🖥️ **Misc & Monitoring**

* `ip neigh` — Show ARP table.
* `arp -a` — View ARP table (legacy).
* `ethtool <iface>` — Show/change Ethernet settings.
* `tcpdump` — Capture packets (requires sudo).
* `wireshark` — GUI-based network analyzer.
* `vnstat` — Show network usage statistics (requires installation).
* `bmon` — Bandwidth monitor in terminal.

---
