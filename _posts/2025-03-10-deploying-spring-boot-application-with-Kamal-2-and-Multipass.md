---
title: "Deploying a Spring Boot Application with Kamal 2 and Multipass"
tags: [tutorial, java, spring boot, docker, deployment]
thumbnail-img: "/assets/img/blog/kamal-multipass-2.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

With **Kamal 2**, you can deploy web apps anywhere—from bare metal to cloud VMs. Kamal offers **zero-downtime deploys, rolling restarts, asset bridging, remote builds, accessory service management**, and everything else you need to deploy and manage your web app in production with **Docker**. Originally built for **Rails** apps, Kamal works with any web app that can be containerized.

**Multipass** provides **Ubuntu VMs on demand** for any workstation. With **cloud-style VMs at your fingertips**, you can launch instances of Ubuntu and initialize them with `cloud-init` metadata in the same way as **AWS, Azure, Google Cloud, IBM, and Oracle**. This allows you to **simulate a cloud deployment** directly on your workstation.

> This post builds upon the concepts discussed in [Advanced Practices in Spring Boot: Building a Modular Application with Docker, Zipkin, and 100% Code Coverage](https://www.makariev.com/blog/advanced-spring-boot-structure-clean-architecture-modulith/).

* toc
{:toc}

## Prerequisites
Before we dive into the development process, ensure you have:

- **Java 21** installed on your system. You can install it using [SDKMAN](https://sdkman.io/usage) and select Java 21.
- **Docker** and **Docker Compose** installed for setting up the local environment.
- Clone the project repository:

```sh
git clone https://github.com/dmakariev/examples.git
cd examples/spring-boot/bookstore
```

## 1. Installing Multipass and Creating VMs
Install Multipass:
```sh
brew install --cask multipass
```

### Step 1: Create Base VM
The `base-vm` will be our **template** for setting up SSH and Docker correctly. We'll use it to clone the other VMs.
```sh
multipass launch -n base-vm --cpus 2 --memory 2G --disk 10G
```

### Step 2: Configure SSH Access in Base VM
Inside the VM:
```sh
multipass shell base-vm
sudo apt update && sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

On your local machine:
```sh
ssh-keygen -t rsa -b 4096
multipass transfer ~/.ssh/id_rsa.pub base-vm:/home/ubuntu/
```

Inside the VM:
```sh
multipass shell base-vm
mkdir -p ~/.ssh
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
rm ~/id_rsa.pub
```

Restart SSH and test access:
```sh
sudo systemctl restart ssh
ssh ubuntu@<base-vm-ip>
```

### Step 3: Install Docker in Base VM
```sh
sudo apt update && sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
newgrp docker
```

Verify Docker installation:
```sh
docker --version
```

### Step 4: Clone VMs
We will now create four additional VMs by cloning `base-vm`:
```sh
multipass clone base-vm --name addon-vm
multipass clone base-vm --name db-vm
multipass clone base-vm --name spring-vm
multipass clone base-vm --name web2-vm
```

Retrieve the list of running VMs:
```sh
multipass list
```
Expected output:
```sh
Name                    State             IPv4             Image
addon-vm                Running           192.168.67.10    Ubuntu 24.04 LTS
base-vm                 Stopped           --               Ubuntu 24.04 LTS
db-vm                   Running           192.168.67.11    Ubuntu 24.04 LTS
spring-vm               Running           192.168.67.9     Ubuntu 24.04 LTS
web2-vm                 Running           192.168.67.12    Ubuntu 24.04 LTS
```

The **spring-vm** and **web2-vm** will act as web instances of our application. In a later blog post, we will cover setting up a **load balancer**, simulating cloud-hosted solutions like **DigitalOcean, AWS Lightsail, and Hetzner**.

## 2. Install Kamal 2
If Ruby is not installed, set it up:
```sh
brew install openssl@3 libyaml gmp rust
curl https://mise.run | sh
echo 'eval "$(~/.local/bin/mise activate)"' >> ~/.zshrc
source ~/.zshrc
mise use -g ruby@3
```

Install Kamal:
```sh
gem install kamal
```

## 3. Build and Push Docker Image
Ensure your Spring Boot application includes the following `LABEL` in the `Dockerfile`:
```Dockerfile
LABEL service="bookstore"
```

Build and push the image to GHCR:
```sh
docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/dmakariev/bookstore:1.0.0 --push .
```

## 4. Configuring Kamal 2
Create `./config/deploy.yml` in the project directory:
```yaml
service: bookstore
image: dmakariev/bookstore

servers:
  web:
    - 192.168.67.9
    - 192.168.67.12

ssh:
  user: ubuntu

proxy:
  ssl: false
  app_port: 80
  healthcheck:
    path: /actuator/health
    interval: 3    # seconds
    timeout: 40    # seconds
```

## 5. Deploying the Application
Run the following commands to deploy:
```sh
kamal setup -P --version=1.0.0
kamal deploy -P --version=1.0.0
```

To update:
```sh
kamal deploy --version=1.1.0
```

## 6. Verifying Deployment
Check logs:
```sh
kamal logs
```

Ensure the application is running:
```sh
curl http://<multipass-vm-ip>/actuator/health
```

## Conclusion
By integrating **Multipass, Docker, and Kamal 2**, we achieve a robust deployment pipeline for a **Spring Boot** application. This approach facilitates local testing in a production-like environment while remaining efficient and manageable. Whether scaling up to a cloud provider or maintaining a local dev setup, this workflow ensures seamless deployment.

In an upcoming post, we will discuss setting up a **load balancer** to distribute traffic across the **spring-vm** and **web2-vm**, simulating cloud-hosted solutions.

Happy coding! 🚀

