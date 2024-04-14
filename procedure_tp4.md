# Reverse proxy
- [Crée et configurer une nouvelle machine virtuelle](#crée-et-configurer-une-nouvelle-machine-virtuelle)
- [Crée la nouvelle alias](#crée-la-nouvelle-alias)
- [Installer Nginx](#installer-nginx)


|                       |             nginx             |            Apache             |             squid             |
| :-------------------: | :---------------------------: | :---------------------------: | :---------------------------: |
| module supplémentaire | :negative_squared_cross_mark: |      :white_check_mark:       | :negative_squared_cross_mark: |
|  simple à configurer  |      :white_check_mark:       | :negative_squared_cross_mark: |      :white_check_mark:       |

Nous allons utiliser nginx dû à sa simplicité et aussi qu'on est déjà familier

Nous nous connectons à dattier

```sh
user@virtu$ ssh virt
```

## Crée et configurer une nouvelle machine virtuelle

Nous créons la machine virtuelle qui sert de _reverse proxy_

```sh
user@virtu$ vmiut creer rproxy
```

Nous allons tout de suite changer son adresse ip
Nous nous connectons en root

On fait les mêmes manipulations qu'à la création de matrix

```sh
user@virtu$ vmiut console rproxy
```
```sh
root@rproxy# ifdown enp0s3
```
```sh
root@rproxy# nano /etc/network/interfaces
```

```sh
iface enp0s3 inet static
        address 10.42.X.2/16
        gateway 10.42.0.1

```

On change le serveur DNS si ce n'est pas le bon

```sh
root@rproxy# nano /etc/resolv.conf
```
```sh
root@rproxy# nameserver 10.42.0.1
```

On peut ensuite redémarrer l'interface

```sh
root@rproxy# ifup enp0s3
```

## Crée la nouvelle alias

On peut quitter la console et configurer la nouvelle alias dans la machine physique pour cela revenons à la machine physique et  dans le _.ssh/config_

```sh
user@phys$ nano .ssh/config
```

Rajoutons 

```sh
Host rproxyjump
        HOSTNAME 10.42.X.2
        User user
        ProxyJump <login>@dattier.iutinfo.fr
        LocalForward phys.iutinfo.fr:9090 localhost:8080  
```

## Installer Nginx

Connectons nous en ssh 
```sh
user@phys$ ssh rproxyjump
```
Mettons nous en root 
```sh
user@rproxy$ su --login
```
Installons nginx
```sh
root@rproxy# apt install nginx
```
Configurons le 
```sh
root@rproxy# nano /etc/nginx/nginx.conf
```
```sh
http {
        server {
                listen localhost:80;
                location / {
                        proxy_pass http://10.42.162.1:80;
                }
        } 
...
```