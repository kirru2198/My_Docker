It's an empty EC2 (Ubuntu) machine

<img width="959" alt="image" src="https://github.com/user-attachments/assets/3d378bc7-235e-4d5f-a93c-0c6a630fb4fc" />

---
default via 172.31.80.1 dev enX0 proto dhcp src 172.31.85.39 metric 100

It means: all outbound traffic will be routed via gateway 172.31.80.1 using interface enX0, with the source IP 172.31.85.39, and the route was assigned via DHCP with a metric of 100.

---

172.31.0.2 via 172.31.80.1 dev enX0 proto dhcp src 172.31.85.39 metric 100

This route directs traffic destined for 172.31.0.2 through gateway 172.31.80.1 via interface enX0, using source IP 172.31.85.39, assigned by DHCP, with a metric of 100.

To reach the IP address 172.31.0.2, send the traffic through 172.31.80.1 using the network interface enX0. The system will use 172.31.85.39 as the source IP. This route was set automatically by DHCP and has a priority (metric) of 100.

---

