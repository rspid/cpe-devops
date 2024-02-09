# TP 01 - Docker

#### Database

Pour build notre Dockerfile nous utilisons la commande en spécifiant en dernier paramètre le chemin de notre fichier
`docker build -t myfirstapp .`

Nous lançons notre container en spécifiant le volume, le network et le nom du build en dernier paramètre.
`docker run --name myfirstapp --net=app-network -v my-vol:/var/lib/postgresql/data myfirstapp`

#### Backend

On utilise un Multistage build premièrement pour éviter de build le code Java sur notre machine mais bien dans un container Docker.

##### Backend API

Après avoir ajouté le Dockerfile et modifié la valeur des champs : url, user et password dans le fichier **application.yml** avec les bonnes valeurs cela fonctionne bien.

#### Http Server

On éxécute la commande : `docker cp <container_name_or_id>:/usr/local/apache2/conf/httpd.conf ./httpd.conf` pour générer le fichier **httpd.conf** sur notre machine pour pouvoir ensuite le modifier.

#### Link

On complète le fichier **docker-compose.yaml** en renseignant les bons ports et les chemins des builds avec les chemins des fichier Dockerfile locaux.

#### Publish

On s'attribue notre image avec la commande `docker tag nom-image-locale davvcpe/nom-image-hébergée:1.0` puis on la publie avec la commande `docker push davvcpe/myhttpd:1.0`.

# TP 02 - Github Actions

Pour finir de compléter le fichier **main.yaml** j'ai suivi le fichier de configuration/build d'un fichier maven donné par Github.

On utilise la commande `mvn package` pour build notre code et la commande `mvn verify` pour lancer la série de tests unitaires.

On va tout d'abord créer un nouveau token sur Dockerhub. Nous allons ensuite créer nos deux repository secrets pour pouvoir ensuite tester notre nouveau pipeline avec la nouvelle action `build-and-push-docker-image`.

Pour le context on renseigne le chemin vers le fichier Dockerfile et pour le tags on renseigne bien le nom de l'image que nous avons push sur Dockerhub.

Pour la suite nous allons générer notre token sur Sonar pour le rajouter sur Github, une fois cela terminée nous n'avons plus qu'a modifier la dernière ligne de vérification pour cette fois utiliser Sonar.

# TP 03 - Ansible

#### Résultat du test de ping avec Ansible

```yaml
david.pichard.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

#### Résultat du test de l'OS

```yaml
david.pichard.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```

#### Résultat de la suppresion de httpd

```yaml
david.pichard.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "",
    "rc": 0,
    "results": [
        "httpd is not installed"
    ]
}
```

#### Résultat de l'éxécution du playbook

```yaml
PLAY [all] ***********************************************************************************************************************************************************************************

TASK [Test connection] ***********************************************************************************************************************************************************************
ok: [david.pichard.takima.cloud]

PLAY RECAP ***********************************************************************************************************************************************************************************
david.pichard.takima.cloud : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Résultat de l'éxécution du playbook advanced

```yaml
PLAY [all] *******************************************************************************************************************************************************************************************

TASK [Install device-mapper-persistent-data] *********************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

TASK [Install lvm2] **********************************************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

TASK [add repo docker] *******************************************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

TASK [Install Docker] ********************************************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

TASK [Install python3] *******************************************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

TASK [Install docker with Python 3] ******************************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

TASK [Make sure Docker is running] *******************************************************************************************************************************************************************
changed: [david.pichard.takima.cloud]

PLAY RECAP *******************************************************************************************************************************************************************************************
david.pichard.takima.cloud : ok=7    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Utilisation des rôles

A la suite de la commande `ansible-galaxy init roles/docker` un dossier roles est bien crée dans notre repository.

On crée ensuite les différents rôles avec les commandes :

```shell
dav@mbad ansible % ansible-galaxy init roles/docker
- Role roles/docker was created successfully
dav@mbad ansible % ansible-galaxy init roles/network
- Role roles/network was created successfully
dav@mbad ansible % ansible-galaxy init roles/database
- Role roles/database was created successfully
dav@mbad ansible % ansible-galaxy init roles/app
- Role roles/app was created successfully
dav@mbad ansible % ansible-galaxy init roles/proxy
- Role roles/proxy was created successfully
dav@mbad ansible %
```

#### Division des tâches

On découpe nos différentes tâches dans des fichiers yaml dans chaque dossier taks de chaque rôle comme cela :

```yaml
# tasks file for roles/backend
- name: Run App
  docker_container:
    name: backend
    image: davvcpe/my-backend:latest
    networks:
      - name: my-network
    state: started
    recreate: true
    pull: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

#### Utilisation de Vault

Une fois cela fait nous allons utilisé Vault pour crypter nos différentes variables d'environnement dans un seul fichier **vault.yml** :

```yaml
POSTGRES_PASSWORD: "pwd"
POSTGRES_USER: "usr"
POSTGRES_DB: "db"
DB_URL: "database:5432"
DB_USER: "usr"
DB_PWD: "pwd"
DB_NAME: "db"
```

Et une fois crypté le fichier ressemble à cela :

```yaml
$ANSIBLE_VAULT;1.1;AES256
35333766633236373462613133303338356463396561613837623336313263376631396634663532
3339316539303836326366323439383665393536626333350a303166646261323732326465663562
62306531656232613966303536363334356632386666366635313231643235613865353262656164
6232356237663530660a393738306464613733303337383939356462633466363266303130396639
32653837386635623261393538393530353933636263386532383636333665393731333938613636
31343537613764646261646665643733316462343035343066313034356139633534303131373862
61373663303565376665343039393536353261303765303739353638363236613866656633643434
66386333666161316561376465656662373963316337666461313262323335336536623263326237
38336637396366616237353531376535633938643666396565643731663932343935666665363733
36383330623964363061356437616639653530623637333430663930323163613138653861636365
636637633937363462666335373863623363
```

Nous devons maintenant renseigner dans notre tâche de déploiement le fichier contenant mes variables d'environnement :

```yaml
---
- hosts: all
  become: true

  roles:
    - docker
    - network
    - database
    - app
    - proxy
  vars_files:
    - vault.yml
```

Nous pouvons maintenant accéder à notre serveur et tester notre api.
