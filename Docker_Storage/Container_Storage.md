In this part of the course, we'll discuss how container storage works in Docker. Understanding storage is essential for using Docker containers effectively. It helps prevent data loss and ensures that containers run as expected. Let's get started!

---

### writable layer

The first type of storage in a container is the writable layer, which I mentioned earlier. To recap, a Docker image consists of three layers, which together form a single file system. When you run a container, a writable layer is added on top, using a union file system. This means you can see all the existing data from the image's layers, and any new or changed data is stored in the writable layer. 

The union file system allows files and directories from different file systems to be combined into one, enabling the writable layer to sit above the read-only layers of the Docker image. This writable layer makes each container unique. However, this architecture requires a file system driver, which can add overhead and affect performance. Keep this in mind when designing containerized applications.

Here are the key points to remember: 

- Layers from the Docker image can be read, but any changes are made in the writable layer. 
- The container views both the image layers and the writable layer as one single file system. 
- The writable layer is tied to the container's lifecycle and cannot be moved elsewhere. 
- Any data written is considered part of that specific container.

