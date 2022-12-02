# TD 6 Mise en production

## Mogenius 

Mongenius est un outil permettant de faciliter la mise en production de notre application. 
Il s'agira ici d'apprhénder les différentes étapes de la mise en production d'un projet via Mogenius. (Petit ou grand projet). En reproduisant les étapes ci-dessous, vous pourrez mettre en production votre projet.

## Prise en main avec la mise en production d'un projet simple

Il s'agira tout d'abord de prendre en main l'outil. Pour ce faire, nous allons essayer de déployer un fichier html simple sur Mongenius.

Pour cela, il faut réussir à build une image docker en local. Voici le docker file à utiliser pour créer une image docker d'un fichier html :

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```


L'image fonctionnant en local, on peut la déployer sur Mogenius. Pour cela, il faut créer un projet sur Mogenius ( après avoir crée un compte ). On peut ensuite lui ajouter des services ( un conteneur ). Ce service peut se base soit sur une template, soit sur un repo github. Il faut cliquer sur le bouton add et choisir l'onglet "Bring your own code". En indiquant le repo github et un port à utiliser, Mogenius va automatiquement pull le repo github et build l'image docker à partir du docker file du repo et la déployer sur le serveur ( Si le repo est public, sinon il faut lier son compte github). On peut ensuite accéder à l'application via l'url fournie par Mogenius en cliquant sur "hostname" puis "External Hostname".

Notre projet contenant un simple fichier html est maintenant déployer. 


## Mise en production d'un projet plus complexe

Il s'agira désormais de déployer un projet plus complexe. Il faut réussir à créer 2 conteneurs et les faires communiquer entre eux ( une base de données avec une api par exemple). Nous allons donc utiliser notre partie back-end du projet fullstack (possédant déjà un dockerfile et un docker compose et donc étant prêt pour la mise en production) et une base PostreSQL.

Voici le lien du dépot github de notre partie back-end pour refaire cette partie si souhaitée : 
https://github.com/Lamkateh/covid-api
Le fichier dockerfile est consultable dans le repo. 

Par le même procédé, il faut ajouter un service à notre projet à partir du même procédé que précédemment. Notre repo sera celui cité précédemment. Il faut faire attention à l'énergie utilisée par ce service (l'énergie est limité car on utilise la partie gratuite). Il faut donc laissé le CPU tel quel, mettre la ram à 50% et le temp.storage à 50% pour avoir et laisser assez d'énergie. 

Il faut ensuite ajouter un service pour la base de données. En cliquant sur le bouton add on peut également ajouter une template. Il permet de déployer un service préconfigurer. 
On va ainsi pouvoir déployer une base de données PostreSQL.


Après avoir créé ces deux services, pour que les conteneurs puissent communiquer entre eux correctement, il faut que l'api puisse communiquer avec la base de données. 
Pour ce faire, dans notre fichier application.yaml de notre api, il y a des varibles d'environnements. Ces variables d'environnements permettent (en les indiquants lors du build de l'image) d'indiquer à l'api les informations de connexion à la base de données.

Il faut donc modifier ces variables d'environnements pour qu'elles correspondent à la base de données déployer sur Mogenius.

Pour modifier ces variables d'environnements sur Mogenius, il faut tout d'abord créer des key vault afin de choisir le nom de la base, le mot de passe et le nom d'utilisateur. Cela permet d'indiquer toute ces informations à Mogenius sans que cela soit en clair dans le code.

Pour ce faire, il faut allez dans l'onglet "Key vault" et ajouter un nouveau key vault ( avec le nom et la valeur souhaité). Voici les key vaults créer : 

![keyvault](/images/keyvault.png)

Ainsi, on peut utiliser ces variables d'environnements dans notre service PostgreSql. Pour ce faire, il faut supprimer et recréer le service (car il faut rebuild avec les variables d'environnements souhaitées). En plus de ces 3 variables, il faut également indiquer le port de la base de données (5432). 

![env_var_postgres](/images/env_var_postgresql.png)


Il faut ensuite modifier les variables d'environnements de notre api. Pour ce faire, il faut aller dans notre service, puis dans l'onglet "ENV VARS" et indiquer les différentes variables d'environnements nécessaires au fonctionnement de notre application et donc présentent dans notre fichier "application.yaml". Dans notre cas, il faut indiquer `DATABASE_HOST` (l'adresse de la base de données, ici se sera l'external hostname de notre service postgresql), `DATABASE_PORT` (le port de la base de données), `DATABASE_PASSWORD` (le mot de passe de la base de données) et `DATABASE_USERNAME` (le nom d'utilisateur de la base de données). On réutilise encore une fois les key vaults créés précédemment.

![env_var_api](/images/env_var_api.png)


Ainsi, notre api est prête à communiquer avec la base de données. Il ne reste plus qu'à tester si cela fonctionne. Pour ce faire, il faut aller dans l'onglet "External Hostname" de notre service api et copier le lien et ajouter "/public/centers". Ceci est une de nos requêtes api. 

Voici notre lien : https://covid-api-prod-mogeniuscoursearmutluethan-t6jelf.mo4.mogenius.io/public/centers

Si tout fonctionne, on devrait observer une réponse json vide.

Voilà comment mettre en production un projet sur Mogenius.




