## Introduction

**GLPI (Gestionnaire Libre de Parc Informatique)** est une application web open source qui permet de gérer l’ensemble des ressources et services informatiques d’une organisation.

C’est un outil de **ITSM (IT Service Management)** et de **gestion de parc** très utilisé par les services informatiques, les administrations et les entreprises de toutes tailles.

GLPI permet de faire : 

- **La gestion de parc informatique (ITAM)**.
- **La gestion des incidents et demandes (Helpdesk / ITIL)**
- **La gestion des utilisateurs et des accès**
- **Une base de connaissances et documentation**

## A) Architecture du projet :

![image.png](attachment:f0ab4f31-3d90-42bf-8e84-fac53f8058dd:image.png)

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

![image.png](attachment:7dc554aa-978d-4a3f-a921-d361f0f2e431:image.png)

Remarque : Le fichier .env n’est pas visible 

### Phase de déploiement de la solution :

```jsx
docker-compose up -d
```

![image.png](attachment:8366e31e-fffa-4d97-ba70-d7b2b6037317:image.png)

- Avec la commande :   docker ps :  on peut voir nos conteneur

![image.png](attachment:707618a6-e854-44b4-8f13-e541d06b40c6:image.png)

- On peut se connecter sur l’interface de notre GLPI :

![image.png](attachment:1eb94047-b6e2-4066-924d-d77231597146:image.png)

Mot de passe par défaut est : glpi/glpi

![image.png](attachment:020ff4d5-add3-4462-83a1-2ae2123e166e:image.png)
