## 🗑️ How to Delete a Specific Docker Image

To delete a specific Docker image by its **image ID** or **name:tag**, use the following command:

```bash
docker rmi <image_id>
```

or

```bash
docker rmi <image_name>:<tag>
```

### 🔍 Example:

```bash
docker rmi my-ubuntu-apache:latest
```

or

```bash
docker rmi 2f2b8fd6b489
```

> ⚠️ Note: Make sure no containers are using the image before deleting it. You may need to stop and remove containers first using `docker rm`.

