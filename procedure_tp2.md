# Installation de PostgreSQL

## Sommaires
- [Installer PostgreSQL](#installer-postgresql)
- [Configurer PostgreSQL](#configurer-postgreql)
- [Crée un utilisateur PosgreSQL](#crée-un-utilisateur-pour-postrgresql)

## Installer Postgresql

Pour installer postgresql utilisé :

```sh
user@matrix$ apt install postgresql postgresql-client
```

Vérifions que le service est en marche

```sh
user@matrix$ systemctl status postgresql
```

Si ce n'est pas le cas effectuer la commande :

```sh
user@matrix$ systemctl enable --now postgresql
```

## Configurer postgreql

Modifions la configuration de connexion

```sh
user@matrix$ nano /etc/postgresql/11/main/pg_hba.conf
```

Modifier la ligne

```sh
local   all             postgres                                peer
```

En

```sh
local   all             postgres                                trust
```

Ainsi que la ligne :

```sh
local   all             all                                peer
```

En

```sh
local   all             all                                md5
```

Ensuite depuis root connecter vous a l'utilisateur postgres

```sh
user@matrix$ su --login
```

```sh
root@matrix# su - postgres
```

Nous allons créer le mot de passe de postgresql

```sh
root@matrix# psql -c "ALTER USER postgres WITH password 'nouveauMotDePasse'"
```

## Crée un utilisateur pour Postrgresql

Nous allons créer un utilisateur de psql matrix avec comme mot de passe matrix

#### Solution 1 :

Crée un utilisateur et modifier son mot de passe

```sh
root@matrix# createuser matrix
```

```sh
root@matrix# psql -c "ALTER USER matrix WITH password 'matrix'"
```

#### Solution 2 :

Crée un utilisateur avec un mot de passe directement

```sh
root@matrix# createuser -P matrix
```

# Installation de Synapse
Pour installer le serveur Synapse nous avons suivi le tutoriel du site [matrix](https://matrix-org.github.io/synapse/latest/setup/installation.html#matrixorg-packages)

## Installation des packages de Synapse
Pour commencer, on installe Synapse

```sh
user@matrix$ sudo apt install -y lsb-release wget apt-transport-https
user@matrix$ sudo wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
```

La commande suivante permet de vérifier si l'installation à bien était effectuer :

```sh
user@matrix$ echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
user@matrix$ sudo tee /etc/apt/sources.list.d/matrix-org.list
```

Puis :

```sh
user@matrix$ sudo apt update
user@matrix$ sudo apt install matrix-synapse-py3
```


Lors de l'installation nous devons entrez le nom de notre instance. Nous devons mettre __machine-physique.iutinfo.fr:8008__.

## Configuration de Synapse

Nous allons maintenant modifier le fichier de configuration de Synapse
Pour cela nous allons faire :

```sh
sudo nano /etc/matrix-synapse/homeserver.yaml
```

Nous allons modifier la ligne __bind_addresses__ en

```sh
bind_addresses: ['::1','0.0.0.0']
```

Car nous utilisons Synapse dans un réseau privée nous devons mettre trusted_key_servers comme ceci

```sh
trusted_key_servers: []
```

Enfin nous ajoutons

```sh
registration_shared_secret: <une chaine de caractères>
```

Nous allons également configurer pour utiliser postgresql au lieu de sqlite

```sh
database:
    name: psycopg2
    args:
        user: matrix
        password: matrix
        database: matrix
        host: localhost
        cp_min: 5
        cp_max: 10
```

## Gérer la base de données de Synapse

On doit crée l'utilisateur et la base de données matrix

```sh
user@matrix$ createuser -P matrix
```

Et ensuite la base de donnée

```sh
user@matrix$ createdb --encoding=UTF8 --locale=C --template=template0 --owner=matrix matrix
```

Nous redémarrons le serveur synapse 

```sh
user@matrix$ sudo systemctl restart matrix-synapse.service
```

Nous pouvons créer les premiers utilisateurs de notre serveur

```sh
user@matrix$ sudo register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml 
```

Notre serveur est opérationnel\
Mais nous allons ajouter le fait de pouvoir créer un compte sur le serveur,
pour cela nous allons modifier encore une fois le fichier homeserver.yaml

```sh
user@matrix$ sudo nano /etc/matrix-synapse/homeserver.yaml
```
Nous allons rajouter les lignes 

```sh
enable_registration : true
enable_registration_without_verification: true
```
La première permet d'autoriser la création de compte
Et la deuxième permet d'autoriser la création de compte sans vérification