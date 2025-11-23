# 🐳 ImageMaid Docker Guide

## Images Docker disponibles

Après le premier push, vos images seront disponibles sur GitHub Container Registry :

```
ghcr.io/nicolasrata/imagemaid:latest          # Dernière version depuis master
ghcr.io/nicolasrata/imagemaid:develop         # Version develop
ghcr.io/nicolasrata/imagemaid:claude-*        # Branches claude
ghcr.io/nicolasrata/imagemaid:v1.2.0          # Tags de version
```

## 🚀 Utilisation rapide

### Option 1 : Docker Run

```bash
docker run --rm \
  -v ~/imagemaid/config:/config:rw \
  -v ~/Library/Application\ Support/Plex\ Media\ Server:/plex:rw \
  ghcr.io/nicolasrata/imagemaid:latest
```

### Option 2 : Docker Compose

Créez `docker-compose.yml` :

```yaml
version: '3.8'

services:
  imagemaid:
    image: ghcr.io/nicolasrata/imagemaid:latest
    container_name: imagemaid
    environment:
      - TZ=Europe/Paris
    volumes:
      - ./config:/config:rw
      - ~/Library/Application Support/Plex Media Server:/plex:rw
```

Puis lancez :
```bash
docker-compose up
```

## 📋 Configuration

Créez `config/.env` :

```bash
PLEX_PATH=/plex
MODE=report
PLEX_URL=http://192.168.1.12:32400
PLEX_TOKEN=votre_token
PHOTO_TRANSCODER=True
PHOTO_TRANSCODER_PATH=/plex/Cache/PhotoTranscoder
LOCAL_DB=False
TRACE=False
```

## 🔄 Mise à jour de l'image

```bash
# Récupérer la dernière version
docker pull ghcr.io/nicolasrata/imagemaid:latest

# Relancer avec la nouvelle image
docker-compose down
docker-compose up
```

## 🛠️ Build local

Si vous voulez construire localement :

```bash
docker build -t nicolasrata/imagemaid:local .
docker run --rm -v ~/imagemaid/config:/config:rw -v ~/Library/Application\ Support/Plex\ Media\ Server:/plex:rw nicolasrata/imagemaid:local
```

## 📅 Exécution planifiée

### Via Docker avec SCHEDULE

Dans votre `.env` :
```bash
SCHEDULE=05:00|weekly(sunday)
```

Puis dans `docker-compose.yml` :
```yaml
services:
  imagemaid:
    image: ghcr.io/nicolasrata/imagemaid:latest
    container_name: imagemaid
    restart: unless-stopped  # Important pour le mode schedule
    volumes:
      - ./config:/config:rw
      - ~/Library/Application Support/Plex Media Server:/plex:rw
```

### Via Cron

```bash
# Éditez le crontab
crontab -e

# Exécution tous les dimanches à 5h
0 5 * * 0 docker run --rm -v ~/imagemaid/config:/config:rw -v ~/Library/Application\ Support/Plex\ Media\ Server:/plex:rw ghcr.io/nicolasrata/imagemaid:latest
```

## 🔍 Debugging

```bash
# Voir les logs
docker logs imagemaid -f

# Accéder au container
docker exec -it imagemaid sh

# Mode debug
echo "TRACE=True" >> config/.env
docker-compose restart
```

## 🏗️ Architecture multi-plateforme

L'image est construite pour :
- `linux/amd64` (Intel/AMD x64)
- `linux/arm64` (Apple Silicon, Raspberry Pi)

Docker choisira automatiquement la bonne architecture.

## 📦 Versions

- `latest` : Version stable depuis master
- `develop` : Version de développement
- `v1.2.0` : Version spécifique
- `claude-*` : Branches de développement Claude

## ⚙️ Variables d'environnement Docker

```yaml
environment:
  - TZ=Europe/Paris              # Timezone
  - PUID=1000                    # User ID (Linux)
  - PGID=1000                    # Group ID (Linux)
```

## 🆘 Dépannage

### L'image ne se télécharge pas

Si l'image est privée, authentifiez-vous :
```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u nicolasrata --password-stdin
```

### Problèmes de permissions

Sur Linux, ajustez PUID/PGID :
```yaml
environment:
  - PUID=1000
  - PGID=1000
```

### L'image est trop ancienne

Forcez le pull :
```bash
docker-compose pull
docker-compose up --force-recreate
```
