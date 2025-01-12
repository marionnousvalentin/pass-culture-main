# pass-culture-main

C'est tout le framework du Pass Culture!

## Minimal Process

### Install

#### Installer les bibliothèques

- Docker
  - [docker](https://docs.docker.com/install/) (testé avec 19.03.12)
  - [docker-compose](https://docs.docker.com/compose/install/#install-compose) (testé avec 1.26.2)
- [NVM](https://github.com/creationix/nvm) (Node Version Manager)
  - `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash`
- [Node](https://nodejs.org/en/download/)
  - `nvm install` dans un répertoire front
- [Yarn](https://yarnpkg.com/lang/fr/docs/install/)
  - Install [HomeBrew](https://brew.sh/index_fr) (MacOS)
  - `brew install yarn --ignore-dependencies` (MacOS)
  - `sudo` (Linux)
- GPG (outil de (dé)chiffrement)
  - [GPG Suite](https://gpgtools.org/) (MacOS)
  - `sudo apt install gpg` (Linux)

#### Pour MacOS spécifiquement

- CoreUtils
  - `brew install coreutils`

#### Installer les CLI

- Netlify
  - `npm install -g netlify-cli@1.2.3`
- Scalingo
  - `curl -O https://cli-dl.scalingo.io/install && bash install`
  - Renseigner ensuite dans l'interface Scalingo la clé SSH publique du device

**Il vous faudra une clé SSH sur votre profil GitHub pour pouvoir cloner le repository.**

#### Installer l'ensemble des projets

1. `git clone git@github.com:pass-culture/pass-culture-main.git pass-culture-main`
2. `cd pass-culture-main`
3. `git submodule update --init --recursive`
4. `./pc symlink`
5. `pc install`

### Lancer les applications

#### API

- `pc start-backend`
- Tester `http://localhost/apidoc/swagger#/default` et vous devriez accéder au swagger
- `pc sandbox -n industrial` (pour peupler la BDD)

#### L'application web

- `pc start-webapp`
- `http://localhost:3000/` devrait être lancé et fonctionnel

- Connectez-vous avec `pctest.jeune93.has-booked-some@example.com` et `user@AZERTY123`

#### Le portail pro

- `pc start-pro`
- `http://localhost:3001/` devrait être lancé et fonctionnel
- Connectez-vous avec `pctest.admin93.0@example.com` et `user@AZERTY123`

### Configurer son API localement

Les actions suivantes sont requises afin de faire fonctionner l'IDE correctement côté API et de pouvoir lancer les `hooks` de `pre-commit`

- Installer Python 3.8
- `cd pass-culture-main/api`
- `python3 -m venv venv`
- `source venv/bin/activate`
- `pip install -r requirements.txt`
- `pip install -e .`
- Installation de nltk:
  - Version Ubuntu: `python -m nltk.downloader punkt stopwords`
  - Version Mac: `pip install nltk`

### Exécution des tests (API, WebApp, Pro)

- Point d'attention : l'API et le pro doivent être lancés pour pouvoir lancer les tests
- API
  1. `pc start-backend`
  2. `pc test-backend` (permet de lancer tous les tests de l'API)
  3. `pc test-backend <path_to_test_file> -k <test_name>` (permet de lancer des tests spécifiques)
  4. `pc test-backend <path_to_test_file> -x <test_name>` (permet de lancer des tests spécifiques et d'arrêter l'exécution au premier test fail)
- WEBAPP / PRO
  1. `yarn test:unit`
  2. `yarn test:cafe` (attention, il doit y avoir des données dans la sandbox au préalable `pc sandbox -n industrial`)

## Développeurs.ses

### Environnement python local

Pour pouvoir lancer les `hooks` de `pre-commit` sur le projet API, il faut installer l'environnement python en local.

- installer Python 3.8 et `pip`
- monter un [virtualenv](https://python-guide-pt-br.readthedocs.io/fr/latest/dev/virtualenvs.html) afin d'avoir un environnement isolé et contextualisé pour les besoins de l'API
  1. `pip install virtualenv`
  2. `cd pass-culture-main/api`
  3. `python -m venv venv`
  4. `source venv/bin/activate`
  5. `pip install -r requirements.txt`

### Rebuild

Pour reconstruire l'image docker sans cache

```bash
pc rebuild-backend
```

### Restart

Pour effacer la base de données complétement, et relancer tous les containers:

```bash
pc restart-backend
```

### Reset

Si vos serveurs de dev tournent, et que vous souhaitez juste effacer les tables de la db:

```bash
pc reset-sandbox-db
```

Si vous voulez juste enlever les recommandations et bookings crées en dev par votre navigation:

```bash
pc reset-reco-db
```

### Migration

Pour effectuer une migration du schéma de la base de données, il est recommandé d'effectuer les étapes suivantes :

1. Modifier le modèle applicatif dans les fichiers python
2. Générer la migration de manière automatique

```
pc alembic  revision --autogenerate -m nom_de_la_migration
```

3. Enlever les commentaires "please adjust" générés par alembic dans le fichier de migration

4. Jouer la migration : `pc alembic upgrade head`

Autres commandes utiles :

- Revenir 1 migration en arrière : `pc alembic downgrade -1`
- Afficher le sql généré entre 2 migrations sans la jouer : `pc alembic upgrade e7b46b06f6dd:head --sql`

### Test

Pour tester le backend:

```bash
pc test-backend
```

Pour tester la navigation du site web

```bash
pc -e production test-cafe-webapp -b firefox
```

Exemple d'une commande test en dev sur chrome pour un fichier test particulier:

```bash
pc test-cafe-pro -b chrome:headless -f signup.js
```

### Restore DB

Pour restorer un fichier de dump postgresql (file.pgdump) en local:

```bash
pc restore-db file.pgdump
```

Pour anonymiser les données après restauration, et changer le mot de passe pour tout les users :

```bash
./api/scalingo/anonymize_database.sh -p PASSWORD
```

### Database de jeu

Afin de naviguer/tester différentes situations de données, il existe dans api des scripts permettant d'engendrer des bases de données "sandbox".

La plus conséquente est l'industrial, elle se créée via la commande:

```bash
pc sandbox -n industrial
```

---

Troubleshooting:

Si la commande sandbox renvoie des erreurs que je n'arrive pas à résoudre, tester de supprimer et reconstruire sa BDD locale. Pour ça:

- stopper les images lancées
- run: `docker rm -f pc-postgres` <= suppression container
- run: `docker volume rm pass-culture-main_postgres_data` <= suppression données
- relancer: `pc start-backend`
- puis relancer: `pc sandbox -n industrial`

---

Cette commande faite, il y a alors deux moyens pour avoir les email/mots de passe des utilisateurs sandbox :

- on peut utiliser la commande sandbox_to_testcafe qui résume les objets utilisés de la sandbox dans les différents testcafés. Si on veut avoir tous les utilisateurs des tests pro_07_offer dans l'application pro, il faut faire:

```
  pc sandbox_to_testcafe -n pro_07_offer
```

- on peut utiliser un curl (ou postman) qui ping directement le server à l'url du getter que l'on souhaite:

```
curl -H "Content-Type: application/json" \
     -H "Origin: http://localhost:3000" \
     GET http://localhost:80/sandboxes/pro_07_offer/get_existing_pro_validated_user_with_validated_offerer_validated_user_offerer_with_physical_venue
```

Il est important que votre serveur API local tourne.

Pour les développeur.ses, quand vous écrivez un testcafé, il faut donc la plupart du temps écrire aussi un getter côté api dans sandboxes/scripts/getters/<moduleNameAvecleMêmeNomQueLeFichierTestcafe>, afin de récupérer les objets souhaités dans la sandbox.

Pour l'application WEBAPP, vous pouvez naviguer avec ce user toujours présent:

```
email: pctest.jeune93.has-booked-some@example.com
```

Pour l'application PRO, vous pouvez naviguer en tant qu'admin avec:

```
email: pctest.admin93.0@example.com
```

Ou en tant qu'user avec :

```
email: pctest.pro93.0@example.com
```

Le mot de passe est toujours : `user@AZERTY123`

(Ces deux utilisateurs existent également pour le 97, pour les utiliser, il suffit de remplacer 93 par 97)

## Tagging des versions

La politique de tagging de version est la suivante :

- On n'utilise pas de _semantic versioning_
- On utilise le format `I.P.S`
  - I => incrément d'**Itération**
  - P => incrément de _fix_ en **Production**
  - S => incrément de _fix_ en **Staging**
- Lors de la pose d'un tag, il faut communiquer les migrations de BDD embarquées à la data pour éviter des bugs sur les analytics

### Exemple

- Je livre une nouvelle version en staging en fin d'itération n°20 => `20.0.0`
- Je m'aperçois qu'il y a un bug en staging => `20.0.1`
- Le bug est corrigé, je livre en production => `20.0.1`
- On détecte un bug en production, je livre en staging => `20.1.0`
- Tout se passe bien en staging, je livre en production => `20.1.0`
- On détecte un autre bug en production, je livre en staging => `20.2.0`
- Je m'aperçois que mon fix est lui-même buggé, je relivre un fix en staging => `20.2.1`
- Mes deux fix sont cette fois OK, je livre en production => `20.2.1`

Pour poser un tag sur une version :

S'assurer d'avoir bien commité ses fichiers.
Checkout de master sur pass-culture-main, pass-culture-api, pass-culture-webapp et pass-culture-pro.

```bash
pc -t I.P.S tag
```

La seule branche devant être taguée de cette façon est master. Pour les hotfixes, voir plus bas.

Le fichier version.txt de l'API est mis à jours ainsi que le package.json de Webapp et Pro.
Le tag est posé sur les branches locales checkout (de préférence master): Api, Webapp et Pro.
Elles sont ensuite poussées sur le repository distant.
Les tests sont enfin joués et on déploie sur Testing.

## Hotfixes

Une fois le commit sur master, déployé en testing et validé par les POs,
pour tagguer les hotfixes, commencer par se placer sur la dernière version déployée
en production ou en staging à l'aide d'un `git checkout vI.P.S` sur chacun de projets.
En effet nous voulons déployer uniquement ce qui est en Prod + nos commits de hotfix.

Une fois le tag checked-out, cherry-pick le fix du bug puis lancer la commande de création de branches de hotfixes et de tag pour chacun des projets :

`pc -t I.P(+1).S(+1) tag-hotfix`.

Une fois les tests de la CI passés, on peut déployer ce tag.
Il faut aussi penser à supprimer les branches de hotfixs une fois le déploiement.

## Deploy

Pré-requis : installer [jq](https://stedolan.github.io/jq/download/)

En premier lieu:

- bien vérifier qu'on a, en local, **main** et tous les submodules **(api, pro, webapp)** à jour par rapport à **master**
- de là on peut poser un tag `pc -t I.P.S. tag` (pour savoir le tag précédent, il suffit de faire un `git tag` dans pass-culture-main)
- se rendre sur CircleCI pour vérifier qu'il y a un job lancé par submodule **(api, pro, webapp)**, ainsi que **main** qui a également lancé autant de jobs qu'il y a de submodules,
- réaliser le déploiement lorsque les tests de chaque submodule sont bien **verts**

Pour déployer une nouvelle version, par exemple en staging:
**(Attention de ne pas déployer sur la production sans concertation !)**

```bash
pc -e <staging|production|integration> -t I.P.S deploy
```

Par exemple pour déployer la version 3.0.1 en integration :

```bash
pc -e integration -t 3.0.1 deploy

```

A la fin de l'opération, une fenêtre de votre navigateur s'ouvrira sur le workflow en cours.

Après avoir livré en production, ne pas oublier de livrer ensuite sur les environnements d'integration.

#### Publier pass-culture-shared sur npm

Pour publier une version de pass-culture-shared sur npm

```bash
cd shared
npm adduser
yarn version
yarn install
npm publish
```

Puis sur webapp et/ou pro, mettre à jour la version de pass-culture-shared dans le fichier `package.json` :

```bash
yarn add pass-culture-shared@x.x.x
git add package.json yarn.lock
```

avec `x.x.x`, étant la nouvelle version déployée sur pass-culture-shared.

## Administration

### Connexion à la base postgreSQL d'un environnement

```bash
pc -e <testing|staging|production> psql
```

ou

```bash
pc -e <testing|staging|production> pgcli
```

### Connexion à la base postgreSQL en local

```bash
pc psql
```

ou

```bash
pc pgcli
```

### Configuration de Metabase

```bash
pc start-metabase
```

Lance Metabase et une base de données contenant les données sandbox du produit.
Pour supprimer les volumes avant de lancer Metabase, utiliser la commande :

```bash
pc restart-metabase
```

L'url pour aller sur Metabase en local est : http://localhost:3002/

Pour configurer Metabase, il suffit de créer un compte admin, puis de se connecter à la base produit. Pour cela, il faut renseigner les informations suivantes :

- Host : pc-postgres-product-metabase
- Port : 5432
- Database name : pass_culture
- Database username : pass_culture
- Database password : passq

### Connexion en ligne de commande python à un environnement (testing | staging | production)

```bash
pc -e <testing|staging|production> python
```

Il est également possible d'uploader un fichier dans l'environnement temporaire grâce à la commande suivante :

```bash
pc -e <testing|staging|production> -f myfile.extension python
```

L'option -f est également disponible pour la commande bash :

```bash
pc -e <testing|staging|production> -f myfile.extension bash
```

Le fichier se trouve à l'emplacement `/tmp/uploads/myfile.extension`

### Acceder au logs des bases de données

En local :

```bash
pc access-db-logs
```

Sur les autres environnements :

```bash
pc -e <testing|staging|production> access-db-logs
```

### Gestion des objects storage OVH

Pour toutes les commandes suivantes, vous devez disposer des secrets de connexion.

Pour lister le contenu d'un conteneur spécifique :

```bash
pc list_content --container=storage-pc-staging
```

Pour savoir si une image existe au sein d'un conteneur :

```bash
pc does_file_exist --container=storage-pc-staging --file="thumbs/venues/SM"
```

Pour supprimer une image au sein d'un conteneur :

```bash
pc delete_file --container=storage-pc-staging --file="thumbs/venues/SM"
```

Pour faire un backup de tous les fichiers du conteneur de production vers un dossier local :

```bash
pc backup_prod_object_storage --container=storage-pc --folder="~/backup-images-prod"
```

Pour copier tous les fichiers du conteneur de production vers le conteneur d'un autre environnement :

```bash
pc copy_prod_container_content_to_dest_container --container=storage-pc-staging
```

## Gestion OVH

#### CREDENTIALS

Vérifier déjà que l'un des admins (comme @arnoo) a enregistré votre adresse ip FIXE (comment connaitre son adresse ip? http://www.whatsmyip.org/)

#### Se connecter à la machine OVH d'un environnement

```bash
pc -e <testing|staging|production> ssh
```

### Dump Prod To Staging

ssh to the prod server

```bash
cd ~/pass-culture-main && pc dump-prod-db-to-staging
```

Then connect to the staging server:

```bash
cd ~/pass-culture-main
cat "../dumps_prod/2018_<TBD>_<TBD>.pgdump" docker exec -i docker ps | grep postgres | cut -d" " -f 1 pg_restore -d pass_culture -U pass_culture -c -vvvv
pc update-db
pc sandbox --name=webapp
```

### Updater le dossier private

Renseigner la variable d'environnement PC_GPG_PRIVATE.
Puis lancer la commande suivante :

```bash
pc install-private
```

#### Updater la db

Une fois connecté:

```
cd /home/deploy/pass-culture-main/ && pc update-db
```

#### Note pour une premiere configuration HTTPS (pour un premier build)

Pour obtenir un certificat et le mettre dans le nginx (remplacer <domaine> par le domaine souhaité, qui doit pointer vers la machine hébergeant les docker)

```bash
docker run -it --rm -v ~/pass-culture-main/certs:/etc/letsencrypt -v ~/pass-culture-main/certs-data:/data/letsencrypt deliverous/certbot certonly --verbose --webroot --webroot-path=/data/letsencrypt -d <domaine>
```

Puis mettre dans le crontab pour le renouvellement :

```bash
docker run -it --rm -v ~/pass-culture-main/certs:/etc/letsencrypt -v ~/pass-culture-main/certs-data:/data/letsencrypt deliverous/certbot renew --verbose --webroot --webroot-path=/data/letsencrypt
```

## Lancer les tests de performance

### Environnement

Les tests requièrent d'avoir un environnement spécifique sur Scalingo, ici `pass-culture-api-perf`, comportant notamment une base utilisateur.
Pour la remplir, il faut jouer les sandboxes `industrial` et `activation`.

Execution des sandboxes sur le conteneur :

```bash
scalingo -a pass-culture-api-perf --region osc-fr1 run 'python src/pcapi/scripts/pc.py sandbox -n industrial'
scalingo -a pass-culture-api-perf --region osc-fr1 run 'python src/pcapi/scripts/pc.py sandbox -n activation'
```

Ensuite, lancer le script d'import des utilisateurs avec une liste d'utilisateurs en csv prédéfinie placée dans le dossier `artillery` sous le nom `user_list`.
On passe en paramètre un faux email qui ne sera pas utilisé.

````bash
scalingo -a pass-culture-api-perf --region osc-fr1 run 'ACTIVATION_USER_RECIPIENTS=<email> python /tmp/uploads/import_users.py user_list' -f scalingo/import_users.py -f user_list```
````

Un exemple de csv utilisateur `user_list` :

```bash
1709,Patricia,Chadwick,ac@enimo.com,0155819967,Drancy (93),16282,2001-05-17,secure_password
```

### Lancement d'un scénario

Pour lancer les tests de performance il faut installer le logiciel `artillery` : `npm install -g artillery` et son plugin `metrics-by-endpoint` : `npm install artillery-plugin-statsd`, puis se munir du fichier csv contenant
les users valides.

Puis se placer dans le dossier `artillery` et lancer la commande :

```bash
artillery run scenario.yml -o reports/report-$(date -u +"%Y-%m-%dT%H:%M:%SZ").json
```

Un rapport des tests daté sera généré dans le sous-dossier `reports` (qui doit être crée).

## Envoyer
