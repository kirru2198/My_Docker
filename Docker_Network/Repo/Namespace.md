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


To create a network namespace, use the command:
```
ip netns add <namespace_name>
```
To list the namespaces, run:
```
ip netns list
```
To see the interfaces on the host, use:
```
ip link
```
To view interfaces within a specific namespace, prefix the command with:
```
ip netns exec <namespace_name> <command>
```
For example, to list interfaces in the "red" namespace:
```
ip netns exec red ip link
```
Inside the namespace, you will only see the loopback interface, not the host's interfaces.

---

**Establishing Connectivity Between Namespaces**

To connect namespaces, we can use a virtual Ethernet pair, which acts like a virtual cable. To create this, run:
```
ip link add <name> type veth peer name <name>
```
Next, attach each interface to the appropriate namespace:
```
ip link set <interface_name> netns <namespace_name>
```
Assign IP addresses to each namespace and bring up the interfaces:
```
ip netns exec <namespace_name> ip addr add <ip_address> dev <interface_name>
ip netns exec <namespace_name> ip link set <interface_name> up
```
Now, the namespaces can communicate with each other.

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
