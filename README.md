#


![Capture d’écran de l’application](Schéma_Architecture.png)

# ⚙️ Architecture du Cluster Docker Swarm

Ce projet met en place une **infrastructure haute disponibilité** reposant sur une architecture modulaire et sécurisée, intégrant **HAProxy**, **Keepalived**, **Traefik v3**, et **Docker Swarm**.

---

## 🧩 Vue d’ensemble

L’infrastructure repose sur :

- 🐳 **3 nœuds Docker Swarm en mode manager** : garantissent le quorum Raft et la haute disponibilité du cluster.
- 🧠 **2 nœuds HAProxy avec Keepalived** : fournissent une IP virtuelle flottante (VIP) pour l’accès externe.
- 🔒 **Terminaison SSL sur HAProxy** : certificats Let’s Encrypt, puis transmission du trafic en TCP vers les nœuds Traefik.
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

## 🚢 Nœuds Managers – Traefik v3 + Socket Proxy

- **Traefik v3** est déployé en mode *global* sur tous les managers.
- Chaque instance communique avec un **socket Docker protégé** via un conteneur `socket-proxy`.
- Le `socket-proxy` limite l’accès à l’API Docker en lecture seule (services, tasks, networks, swarm).
- URL d’accès :  
