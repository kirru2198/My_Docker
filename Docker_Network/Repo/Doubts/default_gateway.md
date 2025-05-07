"If I haven't manually configured any routes inside the container, how is it still able to access the internet?"

---

Great question! Even if you **don't explicitly configure routing** inside a Docker container, it **can still access the internet** because **Docker sets up all the necessary networking automatically** when you run a container in **bridge mode** (which is the default).

Here’s how this works under the hood:

---

### 🔧 1. **Docker Bridge Mode Sets Default Gateway Automatically**

When you start a container, Docker:

* Connects it to the `docker0` bridge.
* Assigns it an IP (e.g., `172.17.0.2`).
* Sets up a **default gateway** inside the container that points to the bridge IP (usually `172.17.0.1`).

You can verify this from inside the container using:

```bash
ip route
```

You’ll typically see something like:

```text
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 scope link
```

So even though **you didn’t configure routing**, Docker did it for you.

---

### 🔁 2. **Docker Host Handles NAT for Internet Access**

* The **Docker host** performs **NAT (masquerading)** using `iptables`.
* When the container sends traffic (e.g., to Facebook), the host changes the **source IP** (from `172.17.0.2`) to the host’s public IP before it goes out.
* The response comes back to the host, which **translates** it back to the container's IP and forwards it inside.

You can see Docker’s NAT rules with:

```bash
sudo iptables -t nat -L -n
```

Look for rules like:

```text
MASQUERADE  all  --  172.17.0.0/16  !172.17.0.0/16  ...
```

---

### 🧠 TL;DR

* Docker automatically sets the **default route** inside containers.
* Docker uses **NAT and the host's network stack** to forward traffic to/from the internet.
* No manual routing is needed unless you’re doing advanced networking.

---



