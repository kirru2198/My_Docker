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

<img width="503" alt="image" src="https://github.com/user-attachments/assets/b74dfc0d-2607-4b04-a2a4-6ae903b8cc55" />

---

<img width="508" alt="image" src="https://github.com/user-attachments/assets/835ff64b-0892-406f-bb70-7d63a97f2481" />

Now, let's explore other storage options available on a single Docker Host with two containers.

The first option is **tmpfs** (or tempfs). This is a file system that uses the host's memory, providing fast, temporary storage. However, it's not persistent because it doesn't save data to disk, and it can't be shared between containers. Use tmpfs for short-term storage, like sensitive files you don't want on disk for long.

The next option is a **bind mount**. This allows you to take a folder from the host's file system (like `/srv`) and mount it inside a container. Multiple containers can access the same folder on the host. This is useful for sharing data stored on the host, but it can reduce portability. For example, a container using a bind mount will only work if the `/srv` folder exists on the host, which might not be the case on different machines. Bind mounts persist beyond the container's lifecycle and can even point to network-based folders.

Lastly, we have **volumes**. These are similar to bind mounts but are managed by Docker. Volumes are created as needed and exist outside the container's lifecycle, so they remain even after the container is removed. They can be moved between containers and attached to multiple containers, but be cautious about multiple processes accessing the same files simultaneously, as there's no file locking.

That's the fundamental overview of storage options. 

