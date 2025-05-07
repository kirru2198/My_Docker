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
ip link delete <interface-name>
```

When you delete one end of a veth pair, the other end is automatically deleted since they are linked.

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

* Attach `veth-red-br` to the bridge (`vnet0`) by setting it as a slave:

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
ip netns exec red ip addr add 192.168.15.1/24 dev veth-red
ip netns exec red ip link set veth-red up

# Inside the blue namespace
ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue
ip netns exec blue ip link set veth-blue up
```

Also, bring up the bridge and host-side interfaces:

```bash
ip link set veth-red-br up
ip link set veth-blue-br up
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

<img width="551" alt="image" src="https://github.com/user-attachments/assets/9d989373-17ca-4f35-b64d-3e9487156aaf" />

