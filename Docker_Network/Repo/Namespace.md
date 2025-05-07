**Introduction**

Today, we will explore **network namespaces** in Linux. 

**What are Namespaces?**

Network namespaces are used by containers, like Docker, to achieve network isolation. Containers are seperated from the Underlying host using namespaces. Think of your host as a house and namespaces as the individual rooms assigned to each child. Each child has privacy in their room and cannot see what happens outside. However, as a parent, you can see all the rooms. Similarly, when you create a container, it is isolated from the host and other containers using namespaces. The container only sees its own processes and thinks it is on its own host, while the host can see everything. (including those running inside containers) 

> The container only sees its own tasks and thinks it is on its own host, while the host can see everything.

**For example:**

- Inside a container, you might see just one process (PID 1).
- From the host, you can see all processes — including those inside the container with a different PID. (same process but different PID inside and outside the container)

This isolation is achieved using namespaces.

<img width="523" alt="image" src="https://github.com/user-attachments/assets/cf757177-f7d5-4e5e-90d6-3bd2f4e6d980" />

---
**Network Namespaces**

When it comes to networking, the host system has its own network interfaces that connect to the local area network (LAN). It also maintains its own routing and ARP (Address Resolution Protocol) tables, which contain information about the rest of the network.

However, when we create a container, we want to isolate it from all this network information. To achieve that, we use a network namespace. A network namespace gives the container its own isolated view of the network.

Within its namespace, the container cannot see the host's network interfaces or routing information. Instead, it has its own virtual network interfaces, its own routing table, and its own ARP table. This setup makes the container feel like it's running on a separate machine with its own network configuration.

<img width="526" alt="image" src="https://github.com/user-attachments/assets/92091b8a-4fef-4516-af8c-29e5eafd7d69" />

---

To create a network namespace, use the command:
```
ip netns add <namespace_name>
```
To list the namespaces, run:
```
ip netns list
```
---

To see the interfaces on the host, use:
```
ip link
```
The Host has loopback interface and eth0 interface 

To view interfaces within a specific namespace, prefix the command with: (This is from the Host)
```
ip netns exec <namespace_name> ip link
```
Or
This only works, if intended to run the command inside the Namespace
```
ip -n <namespace_name> link
```
output:
It only list the loopback interface, you cannot see the eth0 interface on the host.  

For example, to list interfaces in the "red" namespace: (This is from Host)
```
ip netns exec red ip link
```
Inside the namespace, you will only see the loopback interface, not the host's interfaces.

<img width="528" alt="image" src="https://github.com/user-attachments/assets/7acd4ce5-6896-46e7-81a3-94fae6964920" />

---
**ARP table**
The Same is True with the ARP table 
<img width="517" alt="image" src="https://github.com/user-attachments/assets/91e7318e-c9e0-4dfb-9fd3-18689bde90b1" />

---
**Routing Table**
The same is True for Routing table 
<img width="526" alt="image" src="https://github.com/user-attachments/assets/4179b07a-3dbc-4d8e-b802-55277252a2f7" />

---
**Establishing Connectivity Between Namespaces**
---

**Network Connectivity**

By default, newly created network namespaces have no network connectivity. They have no interfaces of their own and cannot see the host's network.

Let’s first establish connectivity **between two namespaces**. Just like we connect two physical machines using Ethernet cables and interfaces, we can connect two network namespaces using a **virtual Ethernet (veth) pair**—think of it as a virtual cable with two ends. This is often referred to as a "pipe", but it's easier to understand as a **virtual cable** with one interface at each end.

To create this virtual cable, we use the following command:

```bash
ip link add veth-red type veth peer name veth-blue
```

This creates two connected interfaces: `veth-red` and `veth-blue`.

<img width="531" alt="image" src="https://github.com/user-attachments/assets/79e0b299-44c9-4420-b28a-662760cfe341" />

Next, we assign each end of the cable to the appropriate network namespace. For example:

```bash
ip link set veth-red netns red
ip link set veth-blue netns blue
```
<img width="531" alt="image" src="https://github.com/user-attachments/assets/23a003f9-ab84-41c6-bd56-445d5ef9289a" />


Now, each namespace has one end of the cable. We can assign IP addresses within the namespaces:

```bash
ip netns exec red ip addr add 192.168.15.1/24 dev veth-red
ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue
```
<img width="527" alt="image" src="https://github.com/user-attachments/assets/2368aaf1-78a7-4206-93ed-d9244f458d17" />

---

Then we bring up the interfaces:
> "enable the interfaces" or "turn on the interfaces."

```bash
ip netns exec red ip link set veth-red up
ip netns exec blue ip link set veth-blue up
```
<img width="530" alt="image" src="https://github.com/user-attachments/assets/a3d2e89a-d6b2-466c-918b-8e6675d0dc63" />

---
An **ARP table** (Address Resolution Protocol table) is a list that your computer or device keeps to match **IP addresses** to **MAC addresses**.

### Why it's needed:

* Computers communicate over a network using IP addresses.
* But on a local network, data is actually delivered using MAC addresses.
* The **ARP table** helps your computer figure out:
  **"What is the MAC address for this IP address?"**

### Example:

If your computer wants to send data to `192.168.1.10`, it checks the ARP table to find the matching MAC address like `00:1A:2B:3C:4D:5E`.

---


At this point, the two namespaces are connected and can communicate with each other. For example, you can run a ping from the red namespace to the blue one:

```bash
ip netns exec red ping 192.168.15.2
```
---

If you inspect the ARP tables, you'll see:

* The red namespace resolves the IP `192.168.15.2` (blue) to its MAC address.
> * The red namespace finds the MAC address of the IP 192.168.15.2 (blue).
> * In networking, a MAC address is a unique identifier assigned to a network interface card (NIC) for communications on the physical network segment.

* The blue namespace resolves `192.168.15.1` (red) to its MAC address.
* The host’s ARP table, however, has no knowledge of these namespaces or their interfaces, because the entire setup is isolated from the host network.

<img width="537" alt="image" src="https://github.com/user-attachments/assets/fc18c8ee-224f-4d80-99a7-3cc97a32b991" />


---

**Creating a Virtual Switch**

When you have multiple containers or namespaces, you need a way for them to communicate with each other—just like devices in the physical world. To enable this communication, you create a **virtual network** inside your host system.

To create a network, you need a **switch**. In the virtual world, that means creating a **virtual switch** within your host. Once the virtual switch is created, you can connect network namespaces to it.

There are several ways to create a virtual switch. Some common solutions include the **Linux bridge** (a native Linux option) and **Open vSwitch (OVS)**. In this example, we’ll use the **Linux bridge**.

First, we create an internal bridge network by adding a new interface to the host using the `ip link add` command with the type set to `bridge`. We'll name this bridge interface **vnet0**. As far as the host is concerned, this is just another network interface—similar to `eth0`. You can see it listed in the output of the `ip link` command alongside other interfaces.

However, by default, the new interface is down. To activate it, use the following command:
```
ip link add name <bridge_name> type bridge
ip link set dev <bridge_name> up
ip link set dev vnet0 up
```
Now, from the perspective of the host, **vnet0** is just another interface. But for the namespaces, **vnet0** acts like a **switch** that they can connect to. So, you can think of it as an interface on the host and a switch for the namespaces.

<img width="543" alt="image" src="https://github.com/user-attachments/assets/019f55b6-23db-4667-9543-54326ca3df10" />

![image](https://github.com/user-attachments/assets/e76fa4b0-02eb-4e92-b6cd-80f55a17b54d)

---

**Connecting to a Virtual Switch**

The next step is to connect the namespaces to the new virtual network switch we created earlier. Previously, we directly connected two namespaces using a virtual Ethernet (veth) pair, where each end of the cable (veth interface) was assigned to a different namespace. But now, since we want **all namespaces to connect to the same bridge network**, we need a new setup.

### 1. Remove the Old Cable

The direct connection (veth pair) between the two namespaces is no longer needed. We can delete it using:

```bash
ip -n red link delete <interface-name>
```

When you delete one end of a veth pair, the other end is automatically deleted since they are linked (since they are a pair).

---

### 2. Create New Cables (veth Pairs)

We will now create **new veth pairs** to connect each namespace to the **Linux bridge (vnet0)**.

For example, for the **red namespace**, run:

```bash
ip link add veth-red type veth peer name veth-red-br
```

* `veth-red`: will go into the **red** namespace.
* `veth-red-br`: will remain in the **host** and connect to the **bridge**.

Use consistent naming (e.g., `veth-blue` and `veth-blue-br`) for other namespaces to make it easy to identify their connections.

---

### 3. Connect the Interfaces

* Attach `veth-red` to the red namespace:

```bash
ip link set veth-red netns red
```

* Attach `veth-red-br` to the bridge (`vnet0`) by setting it as a slave (Here, slave is red):

```bash
ip link set veth-red-br master vnet0
```

Repeat the same steps for the blue namespace (and any others):

```bash
ip link add veth-blue type veth peer name veth-blue-br
ip link set veth-blue netns blue
ip link set veth-blue-br master vnet0
```

---

### 4. Assign IP Addresses and Bring Up Interfaces

Inside each namespace, assign IP addresses to the veth interfaces. Use the same subnet for all:

```bash
# Inside the red namespace
ip netns red ip addr add 192.168.15.1 dev veth-red

# Inside the blue namespace
ip netns blue ip addr add 192.168.15.2/24 dev veth-blue
```

Also, bring up the bridge and host-side interfaces:

```bash
ip -n red link set veth-red up
ip -n blue link set veth-blue up
ip link set vnet0 up
```
---

### 5. Connect Remaining Namespaces

Follow the same procedure to connect the remaining two namespaces (e.g., **green** and **yellow**) to the `vnet0` bridge, assigning them IP addresses like:

* `192.168.15.3`
* `192.168.15.4`

---

### Final Result

All four namespaces are now connected to the internal Linux bridge `vnet0` and can communicate with each other over the **192.168.15.0/24** network:

* red: `192.168.15.1`
* blue: `192.168.15.2`
* green: `192.168.15.3`
* yellow: `192.168.15.4`

They behave just like physical hosts connected through a common switch.

<img width="544" alt="image" src="https://github.com/user-attachments/assets/7551e8c5-c8aa-424a-b89b-39333d7feb57" />

---

Earlier, we assigned our **host** the IP address `192.168.1.2`. Now, what happens if we try to reach one of the interfaces inside the **namespaces** from the host?

At this point, the **host** is on one network (`192.168.1.0/24`), while the **namespaces** are on another (`192.168.15.0/24`). So by default, the host **cannot** communicate with the namespaces.

However, remember that the **bridge (vnet0)** we created acts as a **network interface on the host**. So we do have an interface on the 192.168.15.X network on our host. Since this just another interface all we need to do is assign an Ip address to it. so we can reach Namespaces through it. (Since it is connected to the `192.168.15.0/24` network, we can enable communication by assigning an IP address from that subnet to the bridge interface.

To do that, run:

```bash
ip addr add 192.168.15.5/24 dev vnet0
ip link set vnet0 up
```

Now, the host has an IP (`192.168.15.5`) on the **same network** as the namespaces. You should now be able to ping any namespace from the host, for example:

```bash
ping 192.168.15.1  # Ping the red namespace
```
In the image you provided, it illustrates how a **Linux bridge** (here labeled `v-net-0`) connects multiple network namespaces (with IPs like `192.168.15.1`, `.2`, `.3`, etc.) into a virtual private network (`192.168.15.0/24`).

### How the host can ping the namespace (private network):

Initially, when the host tries to ping `192.168.15.1`, it fails with “Not Reachable!” — this is because the host doesn’t have an IP in the same subnet (`192.168.15.0/24`), so it cannot communicate with that virtual network.

Then, the following command is run:

```bash
ip addr add 192.168.15.5/24 dev v-net-0
```

This adds an IP address (`192.168.15.5`) from the same subnet to the `v-net-0` interface on the host, making it a participant in the virtual bridge.

After this, the host is able to successfully ping `192.168.15.1` because:

* Both the host and the namespace are now in the same subnet.
* They are connected through the Linux bridge (`v-net-0`).

### Summary:

To enable the host to ping a container/namespace in a private network:

1. Assign an IP from the namespace subnet to the bridge interface on the host using:

   ```bash
   ip addr add <bridge-IP>/24 dev <bridge-name>
   ```
2. Ensure routing and firewall rules don’t block the traffic.

---

### Important Notes

This entire setup is still **internal to the host**—meaning it's a **private, isolated network**. From **within the namespaces**, you cannot reach the **external internet**. Likewise, **external systems** cannot reach any applications or services running inside the namespaces.

The **only gateway** to the outside world is the host's physical Ethernet interface (e.g., `eth0`), which is connected to the external network (`192.168.1.0/24`).

To allow internet access for the namespaces, you would need to configure **NAT (Network Address Translation)** or use **IP forwarding**.

<img width="533" alt="image" src="https://github.com/user-attachments/assets/cd8d4424-cde1-4ad8-8ed2-5f3498a4e793" />

---

### Connecting the Bridge Network to the LAN

Let’s say you want the **namespaces**, which are on a private network (`192.168.15.0/24`), to reach another host on your **LAN**—for example, a device with the IP address `192.168.1.3`.

Your **host system** has two interfaces:

* One connected to the private bridge network (`192.168.15.5`)
* Another connected to the external LAN (`192.168.1.2`)

Now let’s say you try to ping `192.168.1.3` from **inside the blue namespace**. Here's what happens:

1. The namespace sees that `192.168.1.3` is not on its local network (`192.168.15.0/24`).
2. It checks its routing table for a way to reach the `192.168.1.0/24` network.
3. Since there is **no route** configured to reach that network, the ping fails with a **"Network unreachable"** error.

---

### Adding a Route to the External Network

To fix this, you need to **add a route** to the routing table in the namespace. But for that, the namespace needs a **gateway**—a device that has access to **both networks**.

So, what acts as the gateway?

Your **host** system!

* It has an interface on the **bridge network** (`192.168.15.5`)
* And it also has an interface on the **LAN** (`192.168.1.2`)

This makes the host a perfect **gateway** between the two networks.

---

### Step-by-Step: Add a Route in the Namespace

From the **blue namespace**, run:

```bash
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
```

This command tells the blue namespace:

> “To reach any IP in the `192.168.1.0/24` network, send packets to the gateway at `192.168.15.5`.”

**Important:**
You must use the **IP address that is reachable from the namespace** (i.e., `192.168.15.5`, not `192.168.1.2`), because the namespace only knows about its local `192.168.15.0/24` network.

---

### Still No Response? Enable NAT on the Host

After adding the route, you may be able to send packets out, but still not receive any replies. Why?

Because your **namespaces use private IP addresses** (like `192.168.15.1`) that the external LAN hosts don’t know how to reply to.

To fix this, you need to enable **NAT (Network Address Translation)** on your host so that it **rewrites the source IP** of packets going out to the LAN, using its own public-facing IP (`192.168.1.2`).

Here’s how you can do that:

1. **Enable IP forwarding**:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

2. **Set up NAT using iptables**:

```bash
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -o eth0 -j MASQUERADE
```

> Replace `eth0` with the actual name of your LAN interface (you can check with `ip link`).

---

### Final Summary

* Your **host** acts as a **gateway** between the namespaces (`192.168.15.0/24`) and the LAN (`192.168.1.0/24`).
* You **add a route** in each namespace pointing to the host’s **bridge IP** as the gateway.
* You **enable NAT** on the host to allow return traffic from the LAN.
* You **enable IP forwarding** so packets can flow between interfaces.

With all this in place, your namespaces can now successfully communicate with external hosts on the LAN like `192.168.1.3`.

<img width="537" alt="image" src="https://github.com/user-attachments/assets/a9587e6b-ac4a-4901-b6f7-7934efdcae0e" />

---
### Enabling NAT for Namespace Internet Access

<img width="535" alt="image" src="https://github.com/user-attachments/assets/ad55bdc2-f33a-4f06-876e-0cebb1e87e22" />
<img width="535" alt="image" src="https://github.com/user-attachments/assets/e4da16c1-1c45-4b66-b578-a7bb9608ba73" />

---

To allow our **namespaces** to reach the **outside world**, we need to enable **NAT (Network Address Translation)** on the **host**. This ensures that any packet leaving the private network appears to originate from the host itself, rather than from inside a namespace.

---

### Step 1: Configure NAT Using `iptables`

We use the `iptables` command to masquerade the source address of packets coming from the namespace network (`192.168.15.0/24`) and going out through the host’s LAN interface (e.g., `eth0`):

```bash
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -o eth0 -j MASQUERADE
```

> Replace `eth0` with your actual LAN interface name (you can find it using `ip link`).

This rule tells the system to **rewrite the source IP** of packets from the private network to the **host’s LAN IP** (e.g., `192.168.1.2`). This way, devices outside your host think the traffic came from the host itself—not from an internal namespace.

---

### Step 2: Enable IP Forwarding

Next, we must enable IP forwarding on the host so that it can route packets between its interfaces:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

To make this change permanent, add the following line to `/etc/sysctl.conf`:

```conf
net.ipv4.ip_forward = 1
```

Then apply it with:

```bash
sysctl -p
```

---

### Step 3: Add a Default Gateway Inside the Namespace

Let’s say you try to ping an **external server** (e.g., `8.8.8.8`) from the **blue namespace**. You may see the message:

```
Network is unreachable
```

Why? Because the namespace's routing table only has a route for the `192.168.1.0/24` network, but **no route to the Internet**.

To fix this, you need to add a **default gateway** in the namespace that points to the host (which acts as the gateway). For example, assuming the host's bridge IP is `192.168.15.5`:

```bash
ip netns exec blue ip route add default via 192.168.15.5
```

This tells the namespace:

> “For any destination not explicitly defined in the routing table, send the traffic to the gateway at `192.168.15.5`.”

---

### Final Result

With the steps above completed:

* The **namespaces** route all outbound traffic to the **host**.
* The **host** uses **NAT** to forward traffic to the Internet.
* The host **replaces the source IP** in outgoing packets, allowing external servers to reply.

As a result, **your namespaces can now access the Internet**, just like any device behind a home router.

---
### External Connectivity to Namespaces

<img width="539" alt="image" src="https://github.com/user-attachments/assets/a12ee3b1-57a6-4964-bfe7-218357f2a451" />

<img width="535" alt="image" src="https://github.com/user-attachments/assets/53541807-09d2-4535-b98d-9180e22fda68" />

---

Let’s now talk about how to allow access from the **outside world** to services running **inside a namespace**.

Imagine your **blue namespace** is hosting a **web application on port 80**. Currently, this namespace is part of a **private internal network** (`192.168.15.0/24`). That means:

* Only the **host system** can access this web application.
* If another machine on the LAN (e.g., `192.168.1.3`) tries to access the namespace's private IP, it will **fail**—because that private IP is **not routable** from outside the host.

---

### Two Options for Enabling External Access

There are two primary ways to enable access to a private namespace from outside:

---

#### **Option 1: Route the Private Network**

You could configure the external host (e.g., `192.168.1.3`) with a **static route**, telling it how to reach the private network.

For example:

```bash
ip route add 192.168.15.0/24 via 192.168.1.2
```

This command means:

> “To reach the private network `192.168.15.0/24`, send packets to `192.168.1.2` (the host system).”

But this method **requires configuration on every external host** and **exposes the internal network**, which is not ideal for security and scalability reasons. So let’s skip this.

---

#### **Option 2: Use Port Forwarding with iptables (Recommended)**

Instead of exposing the entire private network, we can **forward a specific port** from the host to the namespace using `iptables`.

For example, to forward traffic coming to port 80 on the **host's LAN IP** (`192.168.1.2`) to the **blue namespace** at `192.168.15.2:80`, use:

```bash
iptables -t nat -A PREROUTING -p tcp -d 192.168.1.2 --dport 80 -j DNAT --to-destination 192.168.15.2:80
iptables -A FORWARD -p tcp -d 192.168.15.2 --dport 80 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
```

This does two things:

1. **PREROUTING rule**: Redirects incoming traffic on port 80 to the namespace IP and port.
2. **FORWARD rule**: Allows that forwarded traffic to actually pass through.

This approach allows you to:

* Keep your internal network private
* Let outside clients access specific services running inside namespaces

---

### Summary

* Namespaces are on a private network and not directly reachable from outside.
* You have two options to allow access:

  1. Add a **static route** to expose the private network (not recommended).
  2. Use **port forwarding** via `iptables` to expose specific services (recommended).
* With **port forwarding**, the host handles incoming traffic and routes it internally to the correct namespace/service.

---
