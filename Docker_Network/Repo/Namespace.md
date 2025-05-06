**Introduction**

Today, we will explore **network namespaces** in Linux. By the end of this section, you'll take a practice test with challenges related to network namespaces in a hands-on lab environment. Before we dive in, don't forget to subscribe to my channel for more videos like this!

**What are Namespaces?**

Network namespaces are used by containers, like Docker, to achieve network isolation. Think of your host as a house and namespaces as the individual rooms assigned to each child. Each child has privacy in their room and cannot see what happens outside. However, as a parent, you can see all the rooms. Similarly, when you create a container, it is isolated from the host and other containers using namespaces. The container only sees its own processes and thinks it is on its own host, while the host can see everything.

**Network Namespaces**

When it comes to networking, the host has its own interfaces and routing tables. To isolate the container from the host's network details, we create a network namespace for it. This way, the container has its own virtual interfaces and routing tables.

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
