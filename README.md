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
ssh USER@XXX.XXX.XXX.XXX
```
où XXX.XXX.XXX.XXX est l'adresse IP de SIGMA. Il faut alors entrer son mot de passe.

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

Si `dupont` veut sauvegarder les fichiers depuis l'ordinateur `monportable`, il faut taper les lignes suivantes :
```
USER=dupont
ORDINAME=monportable
```

puis coller la ligne suivante dans le terminal:
```
sudo zfs create data/$USER/$ORDINAME && sudo chown -R $USER:$USER /data/$USER && echo "Tout a bien fonctionne. --Personal HPC"
```


Moi, utilisateur, je veux sauvegarder mon ordi. Comment faire ?
---------------------------------------------------------------

Si mon nom d'utilisateur est `dupont` et que mon ordinateur s'appelle `monportable`, il faut d'abord demander à l'administrateur si j'ai bien, moi dupont, un compte pour kemour sur la machine de sauvegarde. Si non, il faudra en créer un. Il faut un compte par utilisateur et par machine : si je veux aussi sauvegarder ma machine nommée `ordi2`, on doit m'ouvrir un autre compte.

Je peux utiliser ma méthode préférée pour faire mes sauvegardes : cp, scp, rsync, ...  
`rsync` est très fortement recommandé. Voici un script bash qui sauvegarde tout mon home :

D'abord, je dois déclarer mon nom d'utilisateur et le nom de l'ordinateur que je veux sauvegarder:
```
  USER=dupont
  ORDINAME=monportable
  HOMEDIR="XXX"
  IPSIGMA="YYY.YYY.YYY.YYY"
```
où `XXX` doit être remplacé par le dossier que vous voulez sauvegarder. `/home/dupont` est recommandé si votre ordinateur est sous linux, ou `/home` ou `/LOCAL` sous macOS.
et `YYY.YYY.YYY.YYY` par l'adresse IP de SIGMA.

puis coller les lignes suivantes dans le terminal:
```
  SOURCE="$HOMEDIR"
  TARGET="$USER@$IPSIGMA:/data/$USER/$ORDINAME"
  OPTIONS="-rtz --progress --delete-after --exclude=.cache"
  rsync $OPTIONS $SOURCE $TARGET
```


Et voilà !




Informations diverses et/ou techniques
=========================================

PHPCsigma135 fonctionne sous Ubuntu server 12.04.4 LTS.
Il y a deux disques durs systèmes en RAID 1, un pour `/`, l'autre pour le swap.
Il y a ensuite 3 zpools : data, databackup, et backupbackup. databackup sauvegarde data. backupbackup sauvegarde databackup.
