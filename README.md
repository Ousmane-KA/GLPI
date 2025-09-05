
---
# GLPI
---

## Introduction

**GLPI (Gestionnaire Libre de Parc Informatique)** est une application web open source qui permet de gérer l’ensemble des ressources et services informatiques d’une organisation.

C’est un outil de **ITSM (IT Service Management)** et de **gestion de parc** très utilisé par les services informatiques, les administrations et les entreprises de toutes tailles.

GLPI permet de faire : 

- **La gestion de parc informatique (ITAM)**.
- **La gestion des incidents et demandes (Helpdesk / ITIL)**
- **La gestion des utilisateurs et des accès**
- **Une base de connaissances et documentation**

## A) Architecture du projet :

<img width="1516" height="577" alt="image" src="https://github.com/user-attachments/assets/0c1560c5-5a0c-4a6a-86a1-acea73f7877d" />


## B) Mise en place de GLPI :

### 1) Configuration du serveur :

- OS : Debian 11
- CPU = 8
- RAM =16
- DISQUE = 150Go
- Installation de docker et docker compose

### 2) déploiement de glpi via docker

- Création du dossier projet :
- Installation des volume conteneur
- implémentation du docker compose : (docker-compose.yml )

```jsx
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: glpi_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/config:/etc/nginx/conf.d:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      - glpi

  glpi:
    image: glpi/glpi:latest
    restart: unless-stopped
    volumes:
      - ./storage/glpi:/var/glpi:rw
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "8080:80"

  db:
    image: mysql
    restart: unless-stopped
    volumes:
      - ./storage/mysql:/var/lib/mysql
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"
      MYSQL_DATABASE: ${GLPI_DB_NAME}
      MYSQL_USER: ${GLPI_DB_USER}
      MYSQL_PASSWORD: ${GLPI_DB_PASSWORD}
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-u", "${GLPI_DB_USER}", "--password=${GLPI_DB_PASSWORD}"]
      start_period: 5s
      interval: 5s
      timeout: 5s
      retries: 10
    expose:
      - "3306"

```

- Implémentation du fichier .env  (.env)

```jsx
GLPI_DB_HOST=db
GLPI_DB_PORT=3306
GLPI_DB_NAME=glpi
GLPI_DB_USER=glpi
GLPI_DB_PASSWORD=glpi

```

- Implémentation du fichier de configuration nginx (glpi.conf )

```jsx
server {
    listen 443 ssl;
    server_name glpi.ka-cloud.com;

    ssl_certificate     /etc/nginx/certs/glpi.crt;
    ssl_certificate_key /etc/nginx/certs/glpi.key;

    location / {
        proxy_pass http://glpi:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

- On va générer un certificat auto-signé :

```jsx
mkdir -p nginx/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/certs/glpi.key \
  -out nginx/certs/glpi.crt

```

- Arborescence du dossier projet :

<img width="1180" height="544" alt="image" src="https://github.com/user-attachments/assets/dfcabc91-6eff-4f81-b23b-a90cc8c18e3e" />


Remarque : Le fichier .env n’est pas visible 

### Phase de déploiement de la solution :

```jsx
docker-compose up -d
```

<img width="1626" height="376" alt="image" src="https://github.com/user-attachments/assets/61bf9ae7-825e-49b6-9186-e993f2f79d3b" />


- Avec la commande :   docker ps :  on peut voir nos conteneur

<img width="1642" height="383" alt="image" src="https://github.com/user-attachments/assets/593ed234-cff8-4f4a-92ff-9f2b0d3d5b44" />


- On peut se connecter sur l’interface de notre GLPI :

<img width="966" height="512" alt="image" src="https://github.com/user-attachments/assets/c47cff25-1852-4e22-8c0f-90b17f7472cb" />


Mot de passe par défaut est : glpi/glpi

<img width="1838" height="372" alt="image" src="https://github.com/user-attachments/assets/6c483897-59ff-4457-8381-ad6abd9cf2d9" />

---
OUSMANE KA

