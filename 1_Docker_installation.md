# Docker Installation on Ubuntu

https://docs.docker.com/engine/install/ubuntu/

Follow the steps below to install Docker on your Ubuntu system.

## 1. Add Docker's Official GPG Key

First, update the package list and install required dependencies:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
```

Create the keyring directory and download Docker's official GPG key:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

## 2. Add Docker Repository to Apt Sources

Add Docker's official repository to your list of Apt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Then update your package list:

```bash
sudo apt-get update
```

## 3. Install Docker

Install Docker, the Docker CLI, containerd, Docker Buildx plugin, and Docker Compose plugin:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## 4. Start and enable Docker service:
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

## 5. Verify Installation

Check the installed version of Docker to verify the installation:

```bash
docker --version
```

## 6. Run the Docker "Hello World" Image

To verify that Docker is working properly, run the `hello-world` image:

```bash
sudo docker run hello-world
```

If everything is set up correctly, you should see a success message confirming that Docker is installed and running.

---
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install the latest version of Docker:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Check the installed Docker version:
docker --version

# Verify that the installation is successful by running the hello-world image:
sudo docker run hello-world
--- 
# use the following commands for automation of the above:
```bash
sudo nano k.sh
```
# copy the above mentioned script inside "k.sh"

# then use the use the following command to execute the file (k.sh)
```bash
sudo bash k.sh
```
