### **Container Data and Read-Only Layers:**
- Containers have a small **write layer** that stores any changes made to the container (like created files).
- This write layer is deleted when the container is removed, explaining why data is lost when the container is deleted.
> - This Write layer depends upon the life-cycle of container.
---

## 5. Introducing Docker Volumes

### **Why Volumes Are Important:**
To preserve data beyond the life of a container, we use **volumes**. Volumes are persistent storage that exist independently of containers. They allow data to survive even after a container is deleted.

- **Think of Volumes as External Storage**: Similar to how a hard drive or a pen drive works, volumes keep your data safe even if the container is deleted.

---

## Example with Volumes

We demonstrated the use of volumes by creating two volumes (`my_wall` and `nginx_log`) and mounting them to containers. Even after deleting the containers, the data in the volumes remained intact.

This is similar to **EBS volumes in EC2**:
- EBS volumes provide persistent storage in the cloud.
- If you terminate an EC2 instance, you can choose to retain the EBS volume, allowing you to attach it to another EC2 instance later.

---

## 11. Data and Backup Considerations

### **Why Backups Are Important:**
- If your local machine crashes, you may lose data unless you use third-party storage solutions (like cloud storage).
- **Cloud Storage**: Storing data remotely ensures that it is safe even if your local machine fails.

### **Backup Policy**:
- A proper backup policy ensures that even if your primary storage system fails, you can recover your data from a backup.

---

<img width="380" alt="image" src="https://github.com/user-attachments/assets/b90583e3-3914-4122-9720-f11f58d30fac" />



