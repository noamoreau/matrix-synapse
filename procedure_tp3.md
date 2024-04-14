# Installation de Synapse

## Sommaires

- [Comparatif de Nginx et Apache](#comparatif-de-nginx-et-apache)
- [Installer le client Element](#installer-le-client-element)
- [Configuration de element](#configuration-de-element)
- [Configuration de Nginx](#configuration-de-nginx)

## Comparatif de Nginx et Apache

|                            |             nginx             |       Apache       |
| :------------------------: | :---------------------------: | :----------------: |
| vitesse de réponse moyenne |           32.848 ms           |     43.419 ms      |
|     contenue dynamique     | :negative_squared_cross_mark: | :white_check_mark: |
|          requêtes          |    plusieurs en même temps    |    une par une     |

Les 2 sont assez similaires mais nous allons partir sur nginx pour plusieurs raisons

1. Il est plus simple a configurer
2. Il est plus rapide

## Installer le client element

Nous allons maintenant installer le client Element

```sh
sudo apt install -y wget apt-transport-https
sudo wget -O /usr/share/keyrings/element-io-archive-keyring.gpg https://packages.element.io/debian/element-io-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/element-io-archive-keyring.gpg] https://packages.element.io/debian/ default main" | sudo tee /etc/apt/sources.list.d/element-io.list
sudo apt update
sudo apt install element-web
```

## Configuration de Element

On configure Element

```sh
user@matrix$ sudo nano /var/www/element-web/config.json
```

```sh
"default_server_config": {
        "m.homeserver": {
            "base_url": "http://phys.iutinfo.fr:8008",
            "server_name": "phys.iutinfo.fr:8008"
        },
        "m.identity_server": {
            "base_url": "https://vector.im"
        }
    },
```

## Configuration de Nginx

On configure ensuite nginx

```sh
user@matrix$ sudo nano /etc/nginx/nginx.conf
```

Dans la partie **http** on met :

```sh
server {
            listen 10.42.X.1:8080;
            root /var/www/element-web/current/;
            location / {
            }
        }

```

On redémarre nginx

```sh
user@matrix$ sudo systemctl restart nginx.service
```

Ensuite sur la machine physique nous rajoutons dans .ssh/config

```sh
user@phys$ sudo nano .ssh/config
```

Dans l'alias **vmjump**

```sh
LocalForward phys.iutinfo.fr:8008 localhost:8008
LocalForward phys.iutinfo.fr:8080 localhost:8080
```

Nous avons maintenant un client Element et un serveur Synapse auto-heberger
