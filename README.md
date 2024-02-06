# TP 01 - Docker

## Database

Pour build notre app nous utilisons la commande
`docker build -t myfirstapp .`

Run notre app en spécifiant le volume et le network.
`docker run --name myfirstapp --net=app-network -v my-vol:/var/lib/postgresql/data myfirstapp`

## Backend API

On utilise un Multistage build premièrement pour éviter de build ce code Java sur notre machine mais bien dans un container Docker.

### Backend API

Après avoir ajouté le Dockerfile et modifié la valeur des champs : url, user et password dans le fichier application.yml avec les bonnes valeurs cela fonctionne.

## Http Server

On éxécute la commande `docker cp <container_name_or_id>:/usr/local/apache2/conf/httpd.conf ./httpd.conf` pour générer le fichier `httpd.conf` sur notre machine pour ensuite le modifier.

## Link

On complète le fichier docker-compose.yaml en renseignant les bons ports et les chemins des builds avec les chemins des fichier Dockerfile locaux.

## Publish

On s'attribue notre image avec la commande `docker tag nom-image-locale davvcpe/nom-image-hébergée:1.0` puis on la publie avec la commande `docker push davvcpe/myhttpd:1.0`.

# TP 02 - Github Actions
