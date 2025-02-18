## Exercice 1 - Conteneurisation avec Docker

### Base de donnÃ©es

**1.1 Pourquoi utiliser des variables d'environnement pour configurer un conteneur ?**

L'utilisation de variables d'environnement permet d'amÃ©liorer la sÃ©curitÃ© et la maintenabilitÃ© des configurations, notamment pour Ã©viter l'exposition de donnÃ©es sensibles.

**1.2 Pourquoi attacher un volume au conteneur PostgreSQL ?**

Les volumes permettent de garantir la persistance des donnÃ©es, facilitant ainsi les sauvegardes et la rÃ©cupÃ©ration des informations en cas de suppression du conteneur.

**1.3 Commandes et Dockerfile de la base de donnÃ©es**

Commande pour exÃ©cuter le conteneur PostgreSQL :
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

![SchÃ©ma Backend](resources/image.png)

Dockerfile :
```yaml
# Ã‰tape de build
FROM maven:3.9.9-amazoncorretto-21 AS build-stage
WORKDIR /app
COPY simpleapi/pom.xml .
COPY simpleapi/src ./src
RUN mvn package -DskipTests

# Ã‰tape de run
FROM amazoncorretto:21
WORKDIR /app
COPY --from=build-stage /app/target/*.jar /app/app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Commande pour dÃ©marrer l'API :
```sh
docker run -p 8080:8080 myapi/image
```

**1.4 Pourquoi utiliser un build multi-Ã©tapes ?**

L'objectif est d'avoir une image optimisÃ©e en sÃ©parant la phase de compilation (Maven) de lâ€™exÃ©cution (JDK). Cela rÃ©duit la taille de l'image finale.

**1.5 Pourquoi un reverse proxy est-il nÃ©cessaire ?**

- SÃ©curise lâ€™accÃ¨s aux services
- Cache certaines requÃªtes pour amÃ©liorer la performance
- Permet la rÃ©partition de charge
- GÃ¨re les certificats SSL/TLS

### Docker Compose

**1.6 Pourquoi docker-compose est-il essentiel ?**

Il simplifie le dÃ©ploiement en dÃ©finissant lâ€™ensemble des services dans un fichier unique, facilitant ainsi la gestion et la collaboration.

**1.7 Commandes Docker Compose essentielles**

- `docker-compose up -d` : Lance lâ€™ensemble des services en arriÃ¨re-plan.
- `docker-compose down -v` : ArrÃªte et supprime les services ainsi que leurs volumes associÃ©s.
- `docker-compose logs -f` : Affiche les logs des conteneurs.
- `docker-compose ps` : VÃ©rifie lâ€™Ã©tat des services.
- `docker-compose exec <service> <commande>` : ExÃ©cute une commande dans un conteneur actif.

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

**2.1 Ã€ quoi servent les testcontainers ?**

Ils permettent dâ€™exÃ©cuter des conteneurs Docker lÃ©gers pour tester des applications en intÃ©gration continue.

![Testcontainers](resources/image-5.png)

**2.2 Configuration de Github Actions**

Les Ã©tapes principales incluent :
- VÃ©rification du code source
- Installation des outils Java/Maven
- ExÃ©cution des tests dâ€™intÃ©gration

**2.3 Pourquoi pousser des images Docker ?**

Cela permet de stocker et de partager des images prÃªtes Ã  Ãªtre dÃ©ployÃ©es sur divers environnements.

![Docker Push](resources/image-6.png)

**2.4 Configuration de la qualitÃ© du code**

- RÃ©cupÃ©ration du code avec `checkout`
- Installation de JDK et des dÃ©pendances
- ExÃ©cution de SonarQube pour lâ€™analyse de la qualitÃ© du code

![Quality Gate](resources/image-7.png)

## Exercice 3 - Automatisation avec Ansible

### Inventaire et commandes de base

**3.1 Fichier dâ€™inventaire et commandes essentielles**

Exemples de commandes :
```sh
ansible all -i inventory/setup.yml -m ping  # VÃ©rifie la connexion SSH
ansible all -i inventory/setup.yml -m apt -a "name=apache2 state=absent" --become  # Supprime Apache
```

### Playbook Ansible

**3.2 Description des tÃ¢ches du playbook**

- Installation des dÃ©pendances
- Configuration du dÃ©pÃ´t Docker
- Installation de Docker et Python
- VÃ©rification du bon fonctionnement de Docker

**3.3 Gestion des conteneurs avec Ansible**

Les rÃ´les dÃ©finissent :
- `network` : crÃ©ation du rÃ©seau `app-network`
- `database` : dÃ©ploiement du conteneur PostgreSQL
- `app` : exÃ©cution de lâ€™API
- `proxy` : mise en place du proxy HTTPD

![DÃ©ploiement Ansible](resources/image-8.png)
