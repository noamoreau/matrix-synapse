# Installation de base

## Sommaires
- [Certaines légendes utile à la procédure](#certaines-légendes-utile-à-la-procédure)
- [Première connexion](#première-connexion)
- [Créer la Machine virtuelle et sa configuration de base](#créer-la-machine-virtuelle-et-sa-configuration-de-base)
- [Configuration de la machinne virtuelle](#configuration-de-la-machinne-virtuelle)
- [Modification pour le confort de travail](#modification-pour-le-confort-de-travail)
- [Dernière configurations pour la VM](#dernière-configurations-pour-la-vm)

## Première connexion

### Connexion avec mot de passe

Assurer vous que vous pouvez vous connecter en ssh a la machine de virtualisation\
Celle-ci est dattier accessible avec _dattier.iutinfo.fr_

```sh
login@phys$ ssh dattier.iutinfo.fr
```

Il vous faut alors votre mot de passe _univ-lille_\
On obtient une empreinte ressemblant à :
```
SHA256:+XNYpzmoYKDnwaB1xqCA2Yu7mBZEK5zvtfXYw1zDO1Y
```
Je confirme l'empreinte après une confirmation faites (Vérifier sur le sujet),\
Cette clé sert d'identifiant pour le ssh

### Connexion sans mot de passe

Pour éviter de devoir réecrire votre mot de passe à chaque connexion :\
Premièrement, nous devons crée une paire de clefs, pour cela entrer le prompt suivant :

```sh
login@phys$ ssh-keygen
```

Vous devriez voir ceci apparaître :

```sh
Generating public/private rsa key pair.
Enter file in which to save the key (/home/infoetu/login/.ssh/id_rsa):
```

Appuyer sur \<ENTER\> pour garder par défaut le fichier dans lequelle la clé va s'écrire, ensuite vous devriez voir

```sh
Enter passphrase (empty for no passphrase):
```

Entrez un mot de passe (garder le en mémoire a chaque première connexion vous allez devoir l'utiliser)\
Par exemple, le même mot de passe que votre compte _univ-lille_\
Un message tel que ci-dessous apparaît :
```sh
SHA256:fMB2KbRIhY0++6x3VcQ8Zmzz9taiJdE79RREr7oPlLw login@phys
The key's randomart image :
+---[RSA 3072]----+
|      .=o    +oo |
|     .o+.. .  @..|
|     .. * o  * +o|
|      oo + ...o.=|
|       oS . +o.=+|
|      .  . .oo= =|
|       o   .E+ + |
|        + . .o   |
|      .o .  ...  |
+----[SHA256]-----+
```
Vous avez réussi a créer votre clé\
Maintenant nous devons envoyer la clé publique au serveur pour l'authentification sans mot de passe :

```sh
ssh-copy-id -i /home/infoetu/login/.ssh/id_rsa.pub  dattier.iutinfo.fr
```

Maintenant essayé de vous reconnecter en ssh

```sh
ssh dattier.iutifo.fr
```

Vous n'avez plus besoin de votre mot de passe pour vous connectez

## Créer la Machine virtuelle et sa configuration de base

Pour créer votre machine virtuelle, connecter vous d'abord en ssh avec l'option -X\
Sans l'option -X, l’application graphique ne peut pas afficher sa fenêtre, l'option -X permet de rediriger une application graphique par la connexion SSH

### Option graphique

```sh
login@phys$ ssh -X dattier.iutinfo.fr
```

Pour crée une vm, sachant que dans notre cas la machine virtuelle ce nommera matrix, utilisez la commande suivante :

```sh
login@dattier$ vmiut creer matrix
```

Ensuite démarrer la VM avec le prompt :

```sh
login@dattier$ vmiut demarrer matrix
```

## Attribué une adresse IP fixe

Chaque fois que vous démarrer la machine virtuelle elle se verra attribuer une adresse ip automatiquement.\
Cela pose problème si l'on veut installer Synapse et y avoir des connexions si l'on redémarre la machine virtuelle.\
Donc nous allons donner une addresse ip fixe à notre machine virtuelle,\
pour cela, sans application graphique (-X), on peut ouvrir la console de la machine virtuelle avec :

```sh
login@dattier$ vmiut console matrix
```

Dans la fenêtre nouvellement apparu, connectez-vous au compte _root_ avec le mot de passe _root_\
Ensuite nous allons désactiver l'interface réseau enp0s3

```sh
root@matrix# ifdown enp0s3
```

Ensuite nous allons modifier le fichier interfaces grâce à nano :

```sh
root@matrix# nano /etc/network/interfaces
```

Vous devez repérer la partie :

```sh
allow-hotplug enp0s3
iface enp0s3 inet dhcp
```

Modifier cette partie du fichier en :
```sh
allow-hotplug enp0s3
iface enp0s3 inet static
        address 10.42.XX.1/16
        gateway 10.42.0.1
```

Grâce à la commande : 
```sh
root@matrix# cat /etc/resolv.conf
```

Vérifier que /etc/resolv.conf est bien comme ceci :
```sh
nameserver 10.42.0.1
```

Ensuite réactiver enp0s3

```sh
ifup enp0s3
```

### Redémarrer la machine virtuelle

#### Solution 1


Sortez de root :
```sh
root@matrix# exit
```
Fermez la page console

Coupez la machinne virtuelle depuis votre machine de virtualisation :
```sh
login@dattier$ vmiut arreter matrix
```
Lancer la machine virtuelle :
```sh
login@dattier$ vmiut demarrer matrix
```

#### Solution 2

Les 3 précédentes commandes peuvent être remplacer par :
```sh
root@matrix# reboot
```

On relance la console :
```sh
login@dattier$ vmiut console matrix
```

On vérifie que l'adresse ip est bien attribué
```sh
root@matrix# ip addr show
```

Vous devriez voir quelque chose comme:
```sh
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ba:79:aa brd ff:ff:ff:ff:ff:ff
    inet 10.42.XX.1/16 brd 10.42.255.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feba:79aa/64 scope link
       valid_lft forever preferred_lft forever

```

Vérifions maintenant que l'addresse du routeur est correct en utilisant le prompt :

```sh
root@matrix# ip route show
```

Vous devriez avoir :

```sh
root@matrix# 10.42.0.0/16 dev enp0s3 proto kernel scope link src 10.42.XX.1
```

Et pour finir vérifions la configuration DNS avec :

```sh
root@matrix# host www.univ-lille.fr
```

XX étant la plage d'adresses IP qui nous a été attribuée devrait voir :

```sh
www.univ-lille.fr is an alias for vip-vs-default.univ-lille.fr.
vip-vs-default.univ-lille.fr has address 194.254.129.23
vip-vs-default.univ-lille.fr has address 194.254.129.22
```
## Configuration de la machinne virtuelle

Les prochaines étapes sont à faire depuis la console

### Installation d'outils

Nous allons installer quelques outils, pour cela nous allons d'abord vérifier/mettre à jour la vm en usant de : 

```sh
root@matrix# apt update && apt full-upgrade
```

Nous allons ensuite redémarrer la vm

```sh
root@matrix# reboot
```

Puis on installe vim, less, tree et rsync

#### Vim

```sh
root@matrix# apt install vim
```

#### Less

```sh
root@matrix# apt install less
```

#### Tree

```sh
root@matrix# apt install tree
```

#### Rsync

```sh
root@matrix# apt install rsync
```

Vous pouvez maintenant quitter la vm et la machine de virtualisation avec

```sh
root@matrix# exit
```

## Modification pour le confort de travail

### Rajout de différents alias

Pour ne plus avoir à passer par la machine de virtuallisation pour accèder à la machine virtuelle comme tels :

```sh
login@phys$ ssh dattier.iutinfo.fr
```

```sh
login@virt$ ssh user@10.42.161.1
```

Nous allons créer des alias\
Pour cela sur la machine physique :

```sh
login@phys$ nano ~/.ssh/config
```

Écrivez

```sh
Host virt
        HostName dattier.iutinfo.fr
        User prenom.nom.etu
        ForwardAgent yes
```

Ceci permet de faire :

```sh
login@phys$ ssh virt
```

Qui correspond à

```sh
login@virt$ ssh dattier.iutinfo.fr
```

Connecter vous à la machine de virtualisation :

```sh
login@phys$ ssh virt
```

Et modiffier le fichier ~/.ssh/config en

```sh
HOST vm
        HOSTNAME 10.42.XX.1
        User user
```

Ceci permet de faire

```sh
login@virt$ ssh vm
```

Ce qui correspond à

```sh
login@virt$ ssh user@10.42.XX.1
```

Revenons sur la machine physique

```sh
login@phys$ exit
```

Ajoutons une nouvelle alias 

```sh
login@phys$ nano ~/.ssh/config
```

```sh
Host vmjump
        HostName 10.42.XX.1
        User user
        ProxyJump prenom.nom.etu@dattier.iutinfo.fr
```

Cette alias remplace

```sh
login@phys$ ssh virt
```

Et

```sh
login@virt$ ssh vm
```

En un simple

```sh
login@phys$ ssh vmjump
```

## Dernière configurations pour la VM

Nous allons changer le nom de la VM
Pour cela connecter vous en root à la vm

```sh
user@matrix$ su --login
```

Ensuite

```sh
root@matrix# hostnamectl set-hostname matrix
```

Puis modifier le fichier /etc/hosts

```sh
root@matrix# nano /etc/hosts
```

Pour que, a la place de __debian__, sur la deuxième ligne, il y est __matrix__.
Maintenant redemarrer la VM

```sh
reboot
```

En vous reconnectant le prompts sera

```sh
user@matrix:~$
```

Et vous pourrez ping matrix grâce à

```sh
user@matrix$ ping matrix
```

Maintenant nous allons donner les droits sudo a l'utilisateur user\
Passons en root

```sh
user@matrix$ su --login
```

Pour cela installons d'abord sudo

```sh
root@matrix# apt install sudo
```

Ensuite nous allons ajouter user au groupe sudo

```sh
root@matrix# usermod -aG sudo user
```

Rédemarrons la VM

```sh
root@matrix# reboot
```

et maintenant en temps que user taper

```sh
user@matrix$ sudo whoami
```

La réponse est __root__, cela montre que vous avez maintenant les droits root