<img width="505" alt="image" src="https://github.com/user-attachments/assets/524eeb9f-1393-42ca-a5bb-a54a49b3593f" />

Welcome back! In this part of the course, we'll cover the basics of Docker Container Networking. This is a complex topic, so we'll keep it simple. 

### Docker Networking Modes

There are two main modes of Docker Networking you need to know: **Host Networking** and **Bridge Networking**.

#### Host Networking

Let's start with Host Networking. Imagine two Docker Hosts, each with two Containers running the same application on TCP Port 1337. In Host Networking, the Containers share the Host's network. This means the Host Port is the same as the Container's application port.

For example, if you run a Container using Port 1337, Heisenberg can access it using the Host's IP and Port 1337. However, if you try to run another Container on the same Host with the same port, it will fail because Port 1337 is already in use. Host Networking works well for different Containers using different ports, but it becomes a problem if you need to run multiple instances of the same Container.

#### Bridge Networking

Bridge Networking solves this issue. With Bridge Networking, a separate bridge network is created, and each Container gets its own unique IP address. This allows multiple Containers to use the same port because their IP addresses are different.

For example, Container one has IP1 on Port 1337, and Container two has IP2 on Port 1337. They can communicate with each other because they are on the same bridge network, but they can't be accessed from outside the Docker Host without publishing their ports.

#### Publishing Containers

To make a Container accessible, you need to publish its port to a Host port, usually written as Host Port:Container Port. For instance, you can run Container one with `-P 1337:1337`, making it accessible to Heisenberg. For Container two, you can't use Port 1337 again, so you might use `-P 1338:1337` instead.

By using Bridge Networking and publishing ports, you can run multiple Containers with the same application on different Host ports.

### Key Takeaways

- **Host Networking**: Simple and requires no configuration, but limits you to one instance of each Container on a Host. You can run many different Containers as long as they use different ports.
- **Bridge Networking**: Overcomes the limitation of Host Networking, but you need to publish port mappings for each Container.

This lesson covers the fundamentals of Docker Networking. There are more advanced options for Bridge Networking and other networking types, which will be discussed in a more in-depth course later. You'll also get practical experience with these concepts soon. For now, that's all on Docker Networking Fundamentals. When you're ready, move on to the next part of the course!
