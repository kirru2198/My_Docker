### 🤔 Why does the container **exit immediately** without `sleep infinity`?

By default, **Docker containers only run as long as their main process is active**. If that process finishes or exits, the container stops.

When you run a container like this:

```bash
sudo docker run -d --name my-container ubuntu
```

It launches the Ubuntu container, but **there’s no long-running process** specified. The container starts, sees there's nothing to do, and exits.

---

### 🛠 What `sleep infinity` does

This command:

```bash
sleep infinity
```

is basically a trick to **keep the container alive forever**. It's a harmless, always-running process. Docker sees that the container has a running process (i.e., `sleep`), so it keeps the container in a running state.

---

### 🧪 Other ways to keep a container running:

1. **Run an interactive shell (only good for foreground testing):**
   ```bash
   sudo docker run -it ubuntu bash
   ```
   But as soon as you exit the shell, the container stops.

2. **Run a real service (e.g., Apache):**
   ```bash
   sudo docker run -d -p 80:80 ubuntu apache2ctl -D FOREGROUND
   ```
   This runs Apache in the foreground, which is the expected mode for Dockerized services.

---

### 🧩 Summary

- Docker containers exit if their main command finishes.
- `sleep infinity` keeps the container running by running a dummy infinite task.
- For production, you should run an actual long-running process (like a server or a daemon).
