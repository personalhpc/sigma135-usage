Comment utiliser le Personal HPC sigma135 
=========================================

Ce site décrit l'utilisation quotidienne du Personal HPC sigma135, un serveur de sauvegarde de 135 TB.  
On ne décrit pas ici comment installer le serveur.  
On appellera cet ordinateur de sauvegarde `SIGMA`.  
Toutes les commandes commençant par `sudo` doivent être executées par un utilisateur ayant les droits d'administration.

Comment éteindre SIGMA proprement ?
-----------------------------------

Par exemple, en cas de coupure de courant prévue,
```
sudo shutdown -h now
```

Comment démarrer SIGMA si elle est éteinte ?
--------------------------------------------

Il y a un bouton (rond) en face avant (photo à venir) qui permet de la démarrer par une simple pression.  
Sous tension, ce bouton devient bleu lumineux.

Que faire pour s'y connecter ?
--------------------------------------------

Il faut se connecter à la machine par ssh avec un utilisateur ayant les droits d'administration :  
```
ssh USER@134.157.169.39
```
Il faut alors entrer son mot de passe.

Comment lister tous les utilisateurs ?
---------------------------------------
```
cat /etc/passwd |grep "/home" |cut -d: -f1
```

Affichera tous les utilisateurs qui ont un dossier home (tous les utilisateurs créés ont par défaut un dossier home). 
```
minimoi
toto
tata
titi
choupli
lola
lili
```


Comment changer le mot de passe d'un utilisateur ?
--------------------------------------------------
De temps en temps, un utilisateur oubliera son mot de passe.
Pour changer le mot de passe de l'utilisateur dupont :
```
sudo passwd dupont
```


Comment ajouter un utilisateur ?
--------------------------------
Si on veut ajouter un utilisateur appelé dupont, il suffit de donner au terminal:
```
USER=dupont
```
puis coller la ligne suivante dans le terminal :
```
sudo adduser $USER && echo "Ce password doit etre transmis a $USER qui doit le changer rapidement"; sudo zfs create data/$USER && sudo chown -R $USER:$USER /data/$USER && echo "Tout a bien fonctionne. --Personal HPC"

```

Comment ajouter un ordinateur à un utilisateur ?
------------------------------------------------

Les utilisateurs doivent avoir un compte pour chaque ordinateur qu'ils voudront sauvegarder.

Si `dupont` veut sauvegarder les fichiers depuis l'ordinateur `kemour`, il faut taper les lignes suivantes :
```
USER=dupont
ORDINAME=kemour
```

puis coller la ligne suivante dans le terminal:
```
sudo zfs create data/$USER/$ORDINAME && sudo chown -R $USER:$USER /data/$USER && echo "Tout a bien fonctionne. --Personal HPC"
```


Moi, utilisateur, je veux sauvegarder mon ordi. Comment faire ?
---------------------------------------------------------------

Si mon nom d'utilisateur est `dupont` et que mon ordinateur s'appelle `kemour`, il faut d'abord demander à l'administrateur si j'ai bien, moi dupont, un compte pour kemour sur la machine de sauvegarde. Si non, il faudra en créer un. Il faut un compte par utilisateur et par machine : si je veux aussi sauvegarder ma machine `hapi`, on doit m'ouvrir un autre compte.

Je peux utiliser ma méthode préférée pour faire mes sauvegardes : cp, scp, rsync, ...  
`rsync` est très fortement recommandé. Voici un script bash qui sauvegarde tout mon home :

D'abord, je dois déclarer mon nom d'utilisateur et le nom de l'ordinateur que je veux sauvegarder:
```
  USER=dupont
  ORDINAME=portmax
  HOMEDIR="/LOCAL"
```

puis coller les lignes suivantes dans le terminal:
```
  SOURCE="$HOMEDIR/$USER/"
  IPSIGMA="134.157.169.39"
  TARGET="$USER@$IPSIGMA:/data/$USER/$ORDINAME"
  OPTIONS="-rtz --progress --delete-after --exclude=.cache"
  rsync $OPTIONS $SOURCE $TARGET
```
Et voilà !




Informations diverses et/ou techniques
=========================================

PHPCsigma135 fonctionne sous CentOS.  
Il y a deux disques durs systèmes en RAID 1, `md0` et `md1`.  
md0 correspond à
  - UUID : c2f02389:83fbf580:7d507bf0:f1a21995
  - /dev/sda1 + /dev/sdb1

md1 correspond à
  - UUID : 2c8158f2:f6df0528:325f979f:e4e96b93
  - /dev/sda2 + /dev/sdb2


On a ensuite un zpool ZFS nommé `data` en raidz3 qui contient 15 disques 3TB: 
créé par un   
```
sudo zpool create -f data raidz3 sdc sdf sdi sdl sdo sdr sdu sdx sdaa sdad sdag sdaj sdam sdap sdas
```

puis un zpool nommé `databackup` qui est dédié à la sauvegarde de data: 
créé par un   
```
sudo zpool create -f databackup raidz3 sdd sdg sdj sdm sdp sds sdv sdy sdab sdae sdah sdak sdan sdaq sdat
```

puis enfin un zpool nommé `backupbackup` qui est dédié à la sauvegarde de la sauvegarde
créé par un   
```
sudo zpool create -f backupbackup raidz3 sde sdh sdk sdn sdq sdt sdw sdz sdac sdaf sdai sdal sdao sdar sdau
```

