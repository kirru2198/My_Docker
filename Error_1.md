### 🛠 Problem:
```bash
Error response from daemon: container <container_id> is not running
```

This error occurs when trying to execute a command inside a Docker container that has already stopped or was never started properly.

---

### ✅ Solution:

#### 1. **Stop the container (if it's still listed)**
```bash
sudo docker stop ubuntu-apache
```

#### 2. **Remove the existing (stopped) container**
```bash
sudo docker rm ubuntu-apache
```

#### 3. **Run a new container in detached mode**
```bash
sudo docker run -d -p 80:80 --name ubuntu-apache ubuntu sleep infinity
```

> 📝 Note: The `sleep infinity` command keeps the Ubuntu container running in the background indefinitely, which allows you to `exec` into it later.

#### 4. **Access the running container**
```bash
sudo docker exec -it ubuntu-apache bash
```
