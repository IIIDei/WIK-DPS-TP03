# WIK-DPS-TP02 - Ping-Api-Dockerized

## Overview

Ce projet est la deuxième étape de notre série de travaux pratiques. Il se base sur le projet [Ping-API](#) qui a permis de créer une API Node.js/TypeScript renvoyant les headers d'une requête GET sur `/ping`. Ici, nous dockerisons cette API en proposant deux approches :

- **Single-Stage Docker Image :** Une image unique intégrant l'application compilée.
- **Multi-Stage Docker Image :** Une image optimisée utilisant une phase de build distincte de l'exécution, ce qui permet de réduire la taille de l'image finale et d'améliorer la sécurité.

De plus, nous utilisons Trivy pour scanner l'image Docker à la recherche de vulnérabilités.

## Contexte et Utilité

- **Objectif :** Dockeriser une application Node.js/TypeScript pour faciliter son déploiement et garantir des builds optimisés.
- **Utilité :** Permettre un déploiement rapide, sécurisé et reproductible de l'API.
- **Sécurité :** Utilisation d'un utilisateur non-root dans le conteneur et scan régulier des images via Trivy.

## Prérequis

### Logiciels requis

- **Node.js** (version 18, comme indiqué dans le fichier `.nvmrc`)
- **npm**
- **Docker**  
  > Sur certaines distributions (ex. Kali Linux), vous devrez peut-être préfixer les commandes Docker avec `sudo`.  
  > Pour éviter cela, ajoutez votre utilisateur au groupe Docker :
  > ```bash
  > sudo usermod -aG docker $(whoami)
  > ```
  > puis déconnectez-vous et reconnectez-vous.
- **Trivy**  
  Un outil de scan des vulnérabilités pour les images Docker.

### Installation des dépendances Node.js

Après clonage du dépôt, installez les dépendances :
```bash
npm install
```

## Compilation & Test Local
1. **Compiler** :
   ```bash
   npm run build
   ```
2. **Lancer l'API** :
   ```bash
   npm start
   ```
3. **Tester** :
   ```bash
   curl -X GET -i http://localhost:8080/ping   # doit renvoyer 200 OK avec JSON
   curl -X POST -i http://localhost:8080/ping   # doit renvoyer 404 Not Found
   ```

## Utilisation de Docker

### Image Single-Stage
- **Build** :
  ```bash
  sudo docker build -t ping-api-single .
  ```
- **Run** :
  ```bash
  sudo docker run -d -p 8080:8080 --name ping-single ping-api-single
  ```
- **Stop & Remove** :
  ```bash
  sudo docker rm -f ping-single
  ```
  
### Image Multi-Stage
- **Build** (sans cache recommandé) :
  ```bash
  sudo docker build --no-cache -f Dockerfile.multi -t ping-api-multi .
  ```
- **Run** :
  ```bash
  sudo docker run -d -p 8080:8080 --name ping-multi ping-api-multi
  ```
- **Stop & Remove** :
  ```bash
  sudo docker rm -f ping-multi
  ```
  
## Scan de Vulnérabilités avec Trivy

### Installation de Trivy (Debian/Kali)
```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.32.1/trivy_0.32.1_Linux-64bit.deb
sudo dpkg -i trivy_0.32.1_Linux-64bit.deb
trivy --version
```

### Scanner l'image
```bash
sudo trivy image ping-api-single
sudo trivy image ping-api-multi
```
