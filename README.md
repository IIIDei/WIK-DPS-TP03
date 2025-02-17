# WIK-DPS-TP03 - Ping-Api-Dockerized with Docker Compose & Reverse-Proxy

## Contexte et Objectifs

Ce projet est la troisième étape de la série de TP. Il s'appuie sur le TP02 (où l'on dockerisait une API Node.js/TypeScript renvoyant les headers d'une requête GET sur `/ping`) et y ajoute :

- **Docker Compose** pour orchestrer l'application entière.
- **4 réplicas** du service API.
- **Un reverse-proxy Nginx** qui est le seul service exposé sur l'hôte (port 8080) et qui fait du load balancing entre les réplicas.
- **Modification de l'API** pour afficher le hostname dans les logs/réponses (pour observer l'équilibrage de charge).

## Prérequis

- **Node.js v18** (cf. `.nvmrc`)
- **npm**
- **Docker**  
  > Utilisez `sudo` si nécessaire ou ajoutez votre utilisateur au groupe Docker :
  > ```bash
  > sudo usermod -aG docker $(whoami)
  > ```
- **Docker Compose**  
  Inclus avec Docker Desktop ou disponible en tant que plugin pour Docker Engine.
- **Trivy** (optionnel) pour scanner les vulnérabilités

## Installation des Dépendances

Après avoir cloné le dépôt, installez les dépendances Node.js :
```bash
npm install
```

## Compilation et Test Local de l'API

1. **Compiler le projet (TypeScript → JavaScript)** :
   ```bash
   npm run build
   ```
2. **Lancer l'API localement** (optionnel) :
   ```bash
   npm start
   ```
3. **Tester l'API** :
   - Requête GET sur `/ping` (doit retourner 200 OK et inclure le hostname) :
     ```bash
     curl -X GET -i http://localhost:8080/ping
     ```
   - Requête POST sur `/ping` (doit retourner 404 Not Found) :
     ```bash
     curl -X POST -i http://localhost:8080/ping
     ```

## Orchestration avec Docker Compose

Le fichier `docker-compose.yaml` orchestre deux services :

- **api** : Construit à partir du Dockerfile (le même que TP02), déployé en 4 réplicas (ne publie pas de port directement).
- **reverse-proxy** : Utilise l'image `nginx:alpine` avec une configuration personnalisée (`nginx.conf`) pour faire du load balancing vers le service API. Seul ce service est exposé sur l'hôte via le port 8080.

### Pour lancer l'environnement complet :

1. **Construire et lancer les services en arrière-plan** :
   ```bash
   sudo docker compose up --build -d
   ```
2. **Vérifier l'état des conteneurs** :
   ```bash
   sudo docker compose ps
   ```
   Vous devriez voir 4 conteneurs pour le service `api` et 1 conteneur pour le service `reverse-proxy`.

3. **Tester le Reverse-Proxy** :
   Envoyez une requête GET sur `/ping` :
   ```bash
   curl -X GET -i http://localhost:8080/ping
   ```
   La réponse doit inclure le hostname du conteneur API qui a traité la requête, démontrant ainsi l'équilibrage de charge.

4. **Arrêter l'environnement** :
   ```bash
   sudo docker compose down
   ```

## Scan de Vulnérabilités avec Trivy (Optionnel)

### Installation de Trivy (Debian/Kali)
```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.32.1/trivy_0.32.1_Linux-64bit.deb
sudo dpkg -i trivy_0.32.1_Linux-64bit.deb
trivy --version
```

### Scanner l'image de l'API
```bash
sudo trivy image wik-dps-tp03-api  # ou l'image construite via votre Dockerfile, si vous souhaitez la scanner séparément
```

## Fichiers Exposés dans le Dépôt

La structure finale du projet est la suivante :
```plaintext
.
├── docker-compose.yaml
├── Dockerfile         # Dockerfile de base pour construire l'image de l'API
├── nginx.conf         # Configuration Nginx pour le reverse-proxy
├── package.json
├── package-lock.json
├── README.md
├── src
│   └── index.ts
└── tsconfig.json

```
