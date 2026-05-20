📡 Homelab Monitoring & Network Lab (VirtualBox + Docker)
🧠 Objectif du projet

Ce projet consiste à créer un homelab de supervision et d’infrastructure réseau basé sur :

VirtualBox
Ubuntu Server
Docker
Grafana
Prometheus
Nginx Proxy Manager

L’objectif est de reproduire une architecture proche d’un environnement professionnel :

serveur central
monitoring système
réseau isolé de lab
accès distant sécurisé via SSH
🏗️ Architecture globale
Windows Host
   |
   | SSH (port 2222)
   v
Ubuntu Server (VM)
   |----------------------|
   | NAT (Internet)       |
   | 10.0.2.15            |
   |                      |
   | Internal Network     |
   | 192.168.50.10        |
   |----------------------|
            |
     Lab Network (isolé)
            |
     +----------------+
     | Kali / VM test |
     +----------------+
⚙️ Étapes d’installation
1. Installation VirtualBox
Installation de VirtualBox
Installation du Extension Pack
Activation VT-x/AMD-V dans le BIOS
2. Création de la VM Ubuntu Server
ISO : Ubuntu Server (version officielle)
RAM : 2 à 4 Go (recommandé 4 Go)
CPU : 2 à 4 vCPU
Disque : 20 à 40 Go (VDI dynamique)
3. Configuration réseau VirtualBox
Carte 1 (NAT)
Accès Internet uniquement
IP automatique (10.0.2.x)
Carte 2 (Internal Network)
Nom : labnet
Réseau isolé pour les VM
🐧 Installation Ubuntu Server

Paramètres :

Langue : English
Partitionnement : par défaut
OpenSSH Server : activé
Hostname : homelab-srv01
🔐 Accès SSH depuis Windows
Configuration VirtualBox (NAT)
Service	Host	Guest
SSH	2222	22

Connexion :

ssh user@127.0.0.1 -p 2222

👉 Cela transforme la VM en serveur distant accessible depuis Windows.

🐳 Installation Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker

docker --version
docker compose version
📊 Stack Monitoring (Prometheus + Grafana)
📌 Rôle de Prometheus

Prometheus est un système de monitoring qui :

collecte des métriques système (CPU, RAM, disque, réseau)
stocke ces données dans une base temporelle
permet l’analyse et les alertes
📌 Rôle de Grafana

Grafana permet :

visualisation des métriques
création de dashboards
suivi en temps réel du système
📁 Structure projet
mkdir -p ~/monitoring/prometheus
cd ~/monitoring
📄 prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
📄 docker-compose.yml
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"

volumes:
  grafana-data:
🚀 Lancement
cd ~/monitoring
docker compose up -d
docker ps
🌐 Accès depuis Windows
Grafana : http://127.0.0.1:3000
Prometheus : http://127.0.0.1:9090

Login Grafana :

user: admin
password: admin
📈 Configuration Grafana
Add Data Source → Prometheus
URL :
http://prometheus:9090
Import dashboard :
ID : 1860 (Node Exporter Full)
🔌 Réseau interne (labnet)

Configuration persistante Ubuntu :

network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.50.10/24
sudo netplan apply
🔀 Reverse Proxy (Nginx Proxy Manager)
Ports VirtualBox
Service	Host	Guest
SSH	2222	22
HTTP	80	80
HTTPS	443	443
NPM Admin	81	81
Accès admin
http://127.0.0.1:81
Configuration DNS local Windows

Fichier :

C:\Windows\System32\drivers\etc\hosts

Ajout :

127.0.0.1 grafana.homelab.local
127.0.0.1 prometheus.homelab.local
Proxy Hosts NPM
Grafana
Domain : grafana.homelab.local
Forward : grafana
Port : 3000
Prometheus
Domain : prometheus.homelab.local
Forward : prometheus
Port : 9090
🔥 Résultat final

Avant :

http://127.0.0.1:3000
http://127.0.0.1:9090

Après :

http://grafana.homelab.local
http://prometheus.homelab.local
🔑 Jump Host SSH

Architecture :

Windows → Ubuntu (SSH)
Ubuntu → autres VM (labnet)

Utilisation :

ssh -J user@127.0.0.1:2222 kali@192.168.50.11
