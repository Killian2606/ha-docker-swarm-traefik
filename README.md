#


![Capture d’écran de l’application](Schéma_Architecture.png)





# ⚙️ Docker Swarm Cluster Architecture

This project sets up a **high-availability infrastructure** based on a modular and secure architecture, integrating **HAProxy**, **Keepalived**, **Traefik v3**, and **Docker Swarm**.

---

## 🧩 Overview

The infrastructure is built on:

- 🐳 **3 Docker Swarm manager nodes** – ensure Raft quorum and cluster high availability.  
- 🧠 **2 HAProxy nodes with Keepalived** – provide a floating Virtual IP (VIP) for external access.  
- 🔒 **SSL termination on HAProxy** – handles certificates and forwards decrypted TCP traffic to Traefik nodes.  
- 🌐 **Traefik v3** – deployed in global mode on managers for dynamic HTTP/S routing.  
- 🧰 **GitLab** – manages configuration (read/write working copy, CI/CD ready for automated updates).  
- ⚙️ **Worker nodes (batch 2)** – dedicated to hosting application services (Nginx, Apache, etc.).

---

## 🖧 Keepalived + HAProxy

- **Keepalived** provides a **Virtual IP (VIP)** shared between two HAProxy instances.  
- In case of an HAProxy node failure, the VIP automatically switches to the other instance.  
- Each HAProxy forwards incoming TCP requests (port `443`) to the **Docker Swarm manager nodes**.  
- **SSL termination** is handled by HAProxy (Let’s Encrypt certificates).  
- HTTPS traffic is decrypted and then forwarded over **TCP** to the Traefik backends.

---


# 🚢 Managers – Traefik v3 + Socket Proxy

This section explains how the manager nodes operate within the Docker Swarm cluster and how to initialize the environment.

---

## ⚙️ Overview

- **Traefik v3** is deployed in *global* mode across all manager nodes.  
- Each instance communicates with a **secured Docker socket** through a `socket-proxy` container.  
- The `socket-proxy` restricts Docker API access to **read-only** (services, tasks, networks, swarm).  
- Access URL:  
tcp://tasks.socket-proxy:2375

yaml


💡 This setup ensures a **centralized and secure view** of the Swarm cluster without exposing the Docker daemon directly.

---

# 🧩 Steps — Docker Swarm Cluster Initialization

Follow the steps below to set up and connect all nodes in your Docker Swarm environment.

---

## 🐳 Step 1 — Initialize the Docker Swarm Cluster

On the **main manager node**, run:

```bash
docker swarm init --advertise-addr 10.10.0.11
Copy the join token displayed at the end of the command:


docker swarm join --token SWMTKN-1-xxxxxxxx 10.10.0.11:2377
On the other manager nodes and the worker, execute the following command to join the cluster:


docker swarm join --token SWMTKN-1-xxxxxxxx 10.10.0.11:2377
To verify that all nodes have successfully joined the cluster:


docker node ls
✅ You should now see all managers and workers listed in the cluster overview.

🌐 Step 2 — Create the Traefik Overlay Network
On any manager node, create the overlay network for Traefik and web services:


docker network create --driver=overlay --attachable traefik
Check that the network has been successfully created:


docker network ls
💡 The traefik overlay network will be used by Traefik and web services to communicate securely across the cluster.

✅ Summary
At this stage:

Your Docker Swarm cluster is initialized and all nodes are connected.

The Traefik overlay network is ready for routing containers and services.

You can now proceed to deploy Traefik v3, HAProxy, and Keepalived for high availability and SSL termination.



