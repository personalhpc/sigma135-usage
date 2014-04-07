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

