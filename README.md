## Exercice 1 - Conteneurisation avec Docker

### Base de données

**1.1 Pourquoi utiliser des variables d'environnement pour configurer un conteneur ?**

L'utilisation de variables d'environnement permet d'améliorer la sécurité et la maintenabilité des configurations, notamment pour éviter l'exposition de données sensibles.

**1.2 Pourquoi attacher un volume au conteneur PostgreSQL ?**

Les volumes permettent de garantir la persistance des données, facilitant ainsi les sauvegardes et la récupération des informations en cas de suppression du conteneur.

**1.3 Commandes et Dockerfile de la base de données**

Commande pour exécuter le conteneur PostgreSQL :
```sh
docker run --net=app-network -v /path/to/data:/var/lib/postgresql/data --name=mydatabase -d mydatabase/image
```

Dockerfile :
```yaml
FROM postgres:14.1-alpine

ENV POSTGRES_DB=mydb \
    POSTGRES_USER=myuser \
    POSTGRES_PASSWORD=mypassword

COPY init.sql /docker-entrypoint-initdb.d
```

### API Backend


Dockerfile :
```yaml
# Étape de build
FROM maven:3.9.9-amazoncorretto-21 AS build-stage
WORKDIR /app
COPY simpleapi/pom.xml .
COPY simpleapi/src ./src
RUN mvn package -DskipTests

# Étape de run
FROM amazoncorretto:21
WORKDIR /app
COPY --from=build-stage /app/target/*.jar /app/app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Commande pour démarrer l'API :
```sh
docker run -p 8080:8080 myapi/image
```

**1.4 Pourquoi utiliser un build multi-étapes ?**

L'objectif est d'avoir une image optimisée en séparant la phase de compilation (Maven) de l’exécution (JDK). Cela réduit la taille de l'image finale.

**1.5 Pourquoi un reverse proxy est-il nécessaire ?**

- Sécurise l’accès aux services
- Cache certaines requêtes pour améliorer la performance
- Permet la répartition de charge
- Gère les certificats SSL/TLS

### Docker Compose

**1.6 Pourquoi docker-compose est-il essentiel ?**

Il simplifie le déploiement en définissant l’ensemble des services dans un fichier unique, facilitant ainsi la gestion et la collaboration.

**1.7 Commandes Docker Compose essentielles**

- `docker-compose up -d` : Lance l’ensemble des services en arrière-plan.
- `docker-compose down -v` : Arrête et supprime les services ainsi que leurs volumes associés.
- `docker-compose logs -f` : Affiche les logs des conteneurs.
- `docker-compose ps` : Vérifie l’état des services.
- `docker-compose exec <service> <commande>` : Exécute une commande dans un conteneur actif.

**1.8 Fichier docker-compose**

```yaml
version: '3.7'
services:
    api:
        build:
          context: ./backend
          dockerfile: Dockerfile
        networks:
          - network_app
        depends_on:
          - db
        ports:
          - "8080:8080"

    db:
        image: postgres:14.1-alpine
        env_file: ".env"
        environment:
          POSTGRES_DB: ${DB_NAME}
          POSTGRES_USER: ${DB_USER}
          POSTGRES_PASSWORD: ${DB_PASS}
        volumes:
          - ./data:/var/lib/postgresql/data
        networks:
          - network_app

    proxy:
        image: httpd:2.4
        container_name: proxy_server
        ports:
          - "80:80"
        volumes:
          - ./proxy/httpd.conf:/usr/local/apache2/conf/httpd.conf
        depends_on:
          - api
        networks:
          - network_app

networks:
    network_app:
      name: network_app
```

## Exercice 2 - CI/CD avec Github Actions

### Testcontainers

**2.1 À quoi servent les testcontainers ?**

Ils permettent d’exécuter des conteneurs Docker légers pour tester des applications en intégration continue.


**2.2 Configuration de Github Actions**

Les étapes principales incluent :
- Vérification du code source
- Installation des outils Java/Maven
- Exécution des tests d’intégration

**2.3 Pourquoi pousser des images Docker ?**

Cela permet de stocker et de partager des images prêtes à être déployées sur divers environnements.


**2.4 Configuration de la qualité du code**

- Récupération du code avec `checkout`
- Installation de JDK et des dépendances
- Exécution de SonarQube pour l’analyse de la qualité du code


## Exercice 3 - Automatisation avec Ansible

### Inventaire et commandes de base

**3.1 Fichier d’inventaire et commandes essentielles**

Exemples de commandes :
```sh
ansible all -i inventory/setup.yml -m ping  # Vérifie la connexion SSH
ansible all -i inventory/setup.yml -m apt -a "name=apache2 state=absent" --become  # Supprime Apache
```

### Playbook Ansible

**3.2 Description des tâches du playbook**

- Installation des dépendances
- Configuration du dépôt Docker
- Installation de Docker et Python
- Vérification du bon fonctionnement de Docker

**3.3 Gestion des conteneurs avec Ansible**

Les rôles définissent :
- `network` : création du réseau `app-network`
- `database` : déploiement du conteneur PostgreSQL
- `app` : exécution de l’API
- `proxy` : mise en place du proxy HTTPD

