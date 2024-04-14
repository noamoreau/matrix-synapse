# Migration vers l’architecture finale

## Sommaires
- [Configuration de bases des 4 machines virtuelles](#configuration-de-bases-des-4-machines-virtuelles)
- [Configurer db](#configurer-db)
- [Configurer matrix](#configurer-matrix)
- [Configurer element](#configurer-element)
- [Configurer rproxy](#configurer-rproxy)


Pour commencer nous allons devoir supprimer matrix 

```sh
user@virtu$ vmiut rm matrix
```

## Configuration de bases des 4 machines virtuelles

Ensuite pour les 4 machines virtuelle : matrix,rproxy,db et element

```sh
user@virtu$ vmiut creer <nom de la machine>
```

```sh
user@virtu$ vmiut start <nom de la machine>
```

```sh
user@virtu$ vmiut console <nom de la machine>
```

Connectez vous en root

On va changer les adresses ip comme suit\
10.42.XX.1 => matrix\
10.42.XX.2 => rproxy\
10.42.XX.3 => db\
10.42.XX.4 => element

```sh
user@vm$ ifdown enp0s3
user@vm$ nano /etc/network/interfaces
```

Changeons le iface enp0s3 en
```sh
allow-hotplug enp0s3
iface enp0s3 inet static
        address 10.42.XX.Y/16
        gateway 10.42.0.1
```
Vous pouvez quitter (en sauvegardant) et ensuite vérifier que __/etc/resolv.conf__ contient nameserver 10.42.0.1

Maintenant faites :

```sh
user@vm$ ifup enp0s3
```

```sh
user@vm$ reboot
```

Une fois ces étapes effectuaient pour les 4 machines virtuelle,\
Revenez a votre machine physique\
et créer les alias pour chacune de ces machines

```sh
user@virtu$ nano /.ssh/config
```

```sh
Host <nom de la machine>jump
    HostName 10.42.X.Y
    User user
    ProxyJump <login>@dattier.iutinfo.fr
```

Ensuite faites :
```sh
user@virtu$ ssh-copy-id <nom de la machine>jump
```

Puis reconnectez-vous à chacune de ces vm:

```sh
user@virtu$ ssh <nom de la machine>jump
```

Enfin connecter vous en root :

```sh
user@vm$ su --login
```

Mot de passe : __root__

Puis

```sh
root@vm# apt update && apt full-upgrade
```

On redémarre les machines virtuelles

```sh
root@vm# reboot
```

Reconnectez-vous, repasser en root puis :

```sh
root@vm# apt install sudo
```

Ensuite donner des droits sudo à l'utilisateur _user_ :

```sh
root@vm# usermod -aG sudo user
```

Pour finir faites de nouveau un reboot\
Reconnecter vous en ssh, ecrivez :

```sh
user@vm$ sudo hostnamectl set-hostname db
```

Nous allons modifier le fichier /etc/hosts depuis la machine de virtualisation

```sh
user@virtu$ nano /etc/hosts
```

```sh
127.0.0.1   localhost
127.0.1.1   <nom de la machine>
10.42.X.Y   <nom des autres machines>
10.42.X.Y   <nom des autres machines> 
10.42.X.Y   <nom des autres machines> 
...
```

Ensuite faites :

```sh
user@virtu$ sudo reboot
```

Vous avez fini la configuration initial des machines

## Configurer db

Maintenant nous allons commencer par configuré db car rproxy a besoin de matrix et d'element,\
element a besoin de matrix , matrix a besoin de db

Connectez vous en ssh

```sh
user@virtu$ ssh dbjump
```

Installons psql

```sh
user@db$ sudo apt install postgresql postgresl-client
```

Configurons le

```sh
user@db$ sudo nano /etc/postgresql/11/main/pg_hba.conf 
```

Faites les changements pour avoir quelque chose comme ceci:

```sh
# DO NOT DISABLE!                                                                 
# If you change this first entry you will need to make sure that the              
# database superuser can access the database using some other method.            
# Noninteractive access to all databases is required during automatic             
# maintenance (custom daily cronjobs, replication, and similar tasks).           #                                                                                 
# Database administrative login by Unix domain socket                            
local   all             postgres                                trust                                                                                               # TYPE  DATABASE        USER            ADDRESS                 METHOD                                                                                              

# "local" is for Unix domain socket connections only                              
local   all             all                                     md5               
# IPv4 local connections:                                                         
host    all             all             127.0.0.1/32            scram-sha-256     
host    matrix          matrix          matrix                  scram-sha-256     
# IPv6 local connections:                                                         
host    all             all             ::1/128                 scram-sha-256     
# Allow replication connections from localhost, by a user with the                
# replication privilege.                                                          
local   replication     all                                     peer              
host    replication     all             127.0.0.1/32            scram-sha-256     
host    replication     all             ::1/128                 scram-sha-256 
```

Ensuite modifions _listen_address_ de /etc/postgresql/15/main/postgresql.conf          

```sh
user@db$ sudo nano /etc/postgresql/15/main/postgresql.conf          
```

```sh
listen_addresses = '*'
```

Redémarrons psql

```sh
user@db$ sudo systemctl restart postgresql.service
```

Nous allons créer l'utilisateur matrix et sa base de données

```sh
user@db$ su --login
```

```sh
root@db# su - postgres
```

```sh
root@db# createuser -P matrix
```

On mets comme mot de passe : __matrix__

```sh
root@db# createdb --encoding=UTF8 --locale=C --template=template0 --owner=matrix matrix 
```

Maintenant nous pouvons quitter db et passer sur matrix

## Configurer matrix

```sh
user@virtu$ ssh matrixjump
```

Installons synapse

```sh
user@matrix$ sudo apt install -y lsb-release wget apt-transport-https
user@matrix$ sudo wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
user@matrix$ echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
sudo tee /etc/apt/sources.list.d/matrix-org.list
user@matrix$ sudo apt update
user@matrix$ sudo apt install matrix-synapse-py3
```

Modifions le fichier de configuration

```sh
user@matrix$ sudo nano /etc/matrix-synapse/homeserver.yaml
```

Pour qu'il soit comme ceci :

```sh
pid_file: "/var/run/matrix-synapse.pid"                                           listeners:                                                                          
   - port: 9090                                                                        
   tls: false                                                                        
   type: http                                                                        
   x_forwarded: true                                                                 
   bind_addresses: ['::1', '0.0.0.0']                                                resources:                                                                         
      - names: [client, federation]                                                      
       compress: false
database:                                                                           
   name: psycopg2                                                                    args:                                                                               
      user: matrix                                                                      
      password: matrix                                                                  
      database: matrix                                                                  
      host: 10.42.162.3                                                                 
      cp_min: 5                                                                         
      cp_max: 10                                                                    
log_config: "/etc/matrix-synapse/log.yaml"                                        
media_store_path: /var/lib/matrix-synapse/media                                   
signing_key_path: "/etc/matrix-synapse/homeserver.signing.key"                    
trusted_key_servers: []                                                           
registration_shared_secret: "secret"                                              
enable_registration : true                                                        
enable_registration_without_verification: true 
```

On redémarre le service

```sh
user@matrix$ sudo systemctl restart matrix-synapse.service
```

On créer un utilsateur

```sh
user@matrix$ sudo register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml
```

Nous en avons fini avec matrix, passons a element

## Configurer element

```sh
user@virtu$ ssh elementjump
```

Nous allons installer element

```sh
user@element$ sudo apt install -y wget apt-transport-https
user@element$ sudo wget -O /usr/share/keyrings/element-io-archive-keyring.gpg https://packages.element.io/debian/element-io-archive-keyring.gpg
user@element$ echo "deb [signed-by=/usr/share/keyrings/element-io-archive-keyring.gpg] https://packages.element.io/debian/ default main" | sudo tee /etc/apt/sources.list.d/element-io.list
user@element$ sudo apt update
user@element$ sudo apt install element-web
```

Nous allons le configurer

```sh
user@element$ sudo nano /var/www/element-web/config.json
```

```sh
"default_server_config": {
        "m.homeserver": {
            "base_url": "http://matrix.phys.iutinfo.fr:8008",
            "server_name": "matrix.phys.iutinfo.fr:8008"
        },
        "m.identity_server": {
            "base_url": "https://vector.im"
        }
    },
...
```

Installons nginx

```sh
user@element$ sudo nano /etc/nginx/nginx.conf
```

On le configure

```sh
http {                                                                                     
    server {                                                                               
        listen 10.42.X.4:9191;                                                           
        root /usr/share/element-web;                                                       
        location / {                                                             
        }                                                                              
    }    
...
```

Maintenant on redémarre nginx

```sh
user@element$ sudo systemctl restart nginx.service 
```

On peut quitter element puis commencer la dernière étape

## Configurer rproxy

```sh
user@virtu$ ssh rproxyjump
```

```sh
user@rproxy$ sudo nano /etc/nginx/nginx.conf  
```

On supprime server pour obtenir ceci

```sh
...
http {                                                                                    
    ##                                                                                 
    # Basic Settings
    ##                                                                                                                                                                    
    sendfile on;                                                                       
    tcp_nopush on;                                                                     
    types_hash_max_size 2048;  
    ...
```

Ensuite dans :

```sh
user@rproxy$ sudo nano /etc/nginx/sites-available/matrix.<phys>.iutinfo.fr 
```

Écrivez

```sh
server {                                                                             
    listen 80;                                                                         
    listen [::]:80;                                                                    
    server_name matrix.<phys>.iutinfo.fr;
    location / {
        proxy_pass http://matrix:9090;
    }                                                                              
}    
```

Puis dans :

```sh
user@rproxy$ sudo nano /etc/nginx/sites-available/element.<phys>.iutinfo.fr 
```

Écrivez

```sh
server {                                                                             
    listen 80;                                                                         
    listen [::]:80;                                                                    
    server_name element.<phys>.iutinfo.fr;
    location / {
        proxy_pass http://element:9191;                                               
    }                                                                                
}    
```

On créer ensuite des liens symboliques

```sh
user@rproxy$ sudo ln -s /etc/nginx/sites-available/matrix.<phys>.iutinfo.fr /etc/nginx/sites-enabled/matrix.<phys>.iutinfo.fr
```

```sh
user@rproxy$ sudo ln -s /etc/nginx/sites-available/element.<phys>.iutinfo.fr /etc/nginx/sites-enabled/element.<phys>.iutinfo.fr
```

On redémarre nginx

```sh
user@rproxy$ sudo systemctl restart nginx.service 
```

On quitte rproxy\
Puis on ajoute un localForward du host rproxyjump

```sh
user@virtu$ nano .ssh/config
```

```sh
Host rproxyjump                                                                            
    HOSTNAME 10.42.X.2                                                              
    User user                                                                          
    ProxyJump <login>@dattier.iutinfo.fr                                        
    LocalForward phys.iutinfo.fr:9090 localhost:80
```

Maintenant on se reconnecte en ssh a rproxy

```sh
ssh rproxyjump
```

Puis vous avez accès à Synapse sur ce lien _http://matrix.<phys>.iutinfo.fr:9090_ et vous avez accès à element _http://element.<phys>.iutinfo.fr:9090_ depuis votre machine physique, tant que la connexion ssh est active.