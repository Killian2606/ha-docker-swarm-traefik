#


![Capture d’écran de l’application](Schéma_Architecture.png)





# ⚙️ Architecture du Cluster Docker Swarm

Ce projet met en place une **infrastructure haute disponibilité** reposant sur une architecture modulaire et sécurisée, intégrant **HAProxy**, **Keepalived**, **Traefik v3**, et **Docker Swarm**.

---

## 🧩 Vue d’ensemble

L’infrastructure repose sur :

- 🐳 **3 nœuds Docker Swarm en mode manager** : garantissent le quorum Raft et la haute disponibilité du cluster.
- 🧠 **2 nœuds HAProxy avec Keepalived** : fournissent une IP virtuelle flottante (VIP) pour l’accès externe.
- 🔒 **Terminaison SSL sur HAProxy** : certificats puis transmission du trafic en TCP vers les nœuds Traefik. 
- 🌐 **Traefik v3** : déployé en mode global sur les managers pour le routage HTTP/S dynamique.
- 🧰 **GitLab** : gestion de la configuration (copie de travail RW, CI/CD possible pour automatiser les mises à jour).
- ⚙️ **Nœuds workers (lot 2)** : prévus pour héberger les services applicatifs (Nginx, Apache, etc.).

---

## 🖧 Keepalived + HAProxy

- **Keepalived** fournit une **VIP** (Virtual IP) entre deux instances HAProxy.  
- En cas de panne d’un nœud HAProxy, la VIP bascule automatiquement vers l’autre.  
- Chaque HAProxy relaie les requêtes TCP entrantes (port `443`) vers les **managers Docker Swarm**.  
- La **terminaison SSL** est effectuée sur HAProxy (certificats Let’s Encrypt).  
- Le trafic HTTPS est déchiffré et relayé en **TCP** vers les backends Traefik.

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


💡 Le réseau traefik sera utilisé par Traefik et les services web pour communiquer à travers le cluster.

