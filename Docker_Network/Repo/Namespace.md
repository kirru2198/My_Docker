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

Next, we assign each end of the cable to the appropriate network namespace. For example:

```bash
ip link set veth-red netns red
ip link set veth-blue netns blue
```

Now, each namespace has one end of the cable. We can assign IP addresses within the namespaces:

```bash
ip netns exec red ip addr add 192.168.15.1/24 dev veth-red
ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue
```

Then we bring up the interfaces:

```bash
ip netns exec red ip link set veth-red up
ip netns exec blue ip link set veth-blue up
```

At this point, the two namespaces are connected and can communicate with each other. For example, you can run a ping from the red namespace to the blue one:

```bash
ip netns exec red ping 192.168.15.2
```

If you inspect the ARP tables, you'll see:

* The red namespace resolves the IP `192.168.15.2` (blue) to its MAC address.
* The blue namespace resolves `192.168.15.1` (red) to its MAC address.
* The host’s ARP table, however, has no knowledge of these namespaces or their interfaces, because the entire setup is isolated from the host network.

---

**Creating a Virtual Switch**

To connect multiple namespaces, we can create a virtual switch using a Linux bridge. Create the bridge with:
```
ip link add name <bridge_name> type bridge
ip link set <bridge_name> up
```
Connect each namespace to the bridge using virtual Ethernet pairs. After setting up the connections, assign IP addresses and bring up the interfaces.

**Connecting to the Host Network**

To allow the host to communicate with the namespaces, assign an IP address to the bridge interface on the host. This way, the host can ping the namespaces.

**Routing to External Networks**

To enable namespaces to reach external networks, add a route in the namespace pointing to the host as the gateway. Use:
```
ip netns exec <namespace_name> ip route add <destination_network> via <gateway_ip>
```
To allow external networks to communicate with the namespaces, you can set up port forwarding using IP tables.
