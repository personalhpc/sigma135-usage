Comment utiliser le Personal HPC sigma135 
=========================================

Ce site décrit l'utilisation quotidienne du Personal HPC sigma135, un serveur de sauvegarde de 135 TB.  
On ne décrit pas ici comment installer le serveur.  
On appellera cet ordinateur de sauvegarde `SIGMA`.  
Toutes les commandes commençant par `sudo` doivent être executées par un utilisateur ayant les droits d'administration.

Comment démarrer SIGMA si elle est éteinte ?
--------------------------------------------

Il y a un bouton (rond) en face avant (photo à venir) qui permet de la démarrer par une simple pression.  
Sous tension, ce bouton devient bleu lumineux.

Que faire après le démarrage ?
------------------------------

D'abord, il faut se connecter à la machine par ssh avec un utilisateur ayant les droits d'administration :  
```
ssh USER@134.157.169.39
```
Il faut alors entrer son mot de passe.

Il faut ensuite redémarrer le système de fichier ZFS  
```
sudo /etc/init.d/zfs.fuse start
```
On pourrait décider de démarrer ZFS automatiquement, mais je crois que c'est une bonne idée.

Ça y est ! Le système est prêt à tourner jusqu'à la prochaine coupure de courant :-)

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
De temps en temps, un utilisateur perdra son mot de passe.
Pour changer le mot de passe de l'utilisateur jeansarkozy :
```
sudo passwd jeansarkozy
```


Comment ajouter un utilisateur ?
--------------------------------
Si on veut ajouter un utilisateur appelé [jeansarkozy](http://fr.wikipedia.org/wiki/Jean_Sarkozy), il suffit de donner au terminal:
```
USER=jeansarkozy
```
puis, sans se soucier de ce que ça veut dire :
```
sudo adduser $USER && sudo zfs create tank/$USER && sudo chown -R $USER:$USER /tank/$USER && sudo passwd $USER
```







Informations diverses et/ou techniques
---------------------------------------

PHPCsigma135 fonctionne sous CentOS.  
Il y a deux disques durs systèmes en RAID 1, `md0` et `md1`.  
md0 correspond à
  - UUID : c2f02389:83fbf580:7d507bf0:f1a21995
  - /dev/sda1 + /dev/sdb1

md1 correspond à
  - UUID : 2c8158f2:f6df0528:325f979f:e4e96b93
  - /dev/sda2 + /dev/sdb2


On a ensuite un zpool ZFS nommé `data` en raidz3 qui contient 15 disques 3TB: 
*c, f, i, l, o, r, u, x, aa, ad, ag, aj, am, ap, as*  
créé par un   
```
sudo zpool create data raidz3 sdc sdf sdi sdl sdo sdr sdu sdx sdaa sdad sdag sdaj sdam sdap sdas
```

puis un zpool nommé `databackup` qui est dédié à la sauvegarde de data: 
*d, g, j, m, p, s, v, y, ab, ae, ah, ak, an, aq, at*  
créé par un   
```
sudo zpool create databackup raidz3 sdd sdg sdj sdm sdp sds sdv sdy sdab sdae sdah sdak sdan sdaq sdat
```

puis enfin un zpool nommé `backupbackup` qui est dédié à la sauvegarde de la sauvegarde
*e, h, k, n, q, t, w, z, ac, af, ai, al, ao, ar, au*    
créé par un   
```
sudo zpool create backupbackup raidz3 sde sdh sdk sdn sdq sdt sdw sdz sdac sdaf sdai sdal sdao sdar sdau
```

