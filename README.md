<p align="right">
<img src="https://user-images.githubusercontent.com/72150651/153272845-88618bfc-e46f-4482-b249-9dc26c550ca1.png"/>
</p  
 
-----------------
# CPE LYON - 4ETI - Année 2022/2023  
## Administration Système  
# __TP7 - Boot, services et processus / Tâches d'administration__  
###### Compte rendu réalisé par Mariem Ben Salah
-----------------
 
# Exercice 1 - Personnalisation de GRUB
 
1. Le fichier `/etc/default/grub/.d/50-curtin-settings.cg` n'existe pas sur ma machine.
 
2. Pour afficher le menu de GRUB pendant 10 secondes, il suffit de modifier le fichier -plus précisemment, deux variables qui existent dans- `/etc/default/grub`.
``` bash
 GRUB_TIMEOUT_STYLE=menu;
 GRUB_TIMEOUT=10;
  
 ```
Dans le cas où la variable __GRUB_TIMEOUT_STYLE__ est mise à 'menu', GRUB va forcer l'affichage du menu et attendre le timeout mis dans la variable __GRUB_TIMEOUT__ pour lancer le premier OS du menu.
 
3.   
 
&#x274C; __ATTENTION__ il faut toujours lancer les commandes `update-grub` et `reboot` après chaque configuration pour valider que les changement ont bien été pris en compte.
 
La commande `update-grub` fait appel au script grub-mkconfig qui construit le fichier de configuration 'final' de GRUB __(/boot/grub/grub.cfg)__ à partir du fichier de paramètres et des scripts.
 
4. Pour changer la résolution de GRUB et de la VM, on procède de la même manière que la question précédente en modifiant cette fois les deux variables suivantes : 
 ``` bash
 GRUB_GFXMODE=1024x768x16
 GRUB_GFXPAYLOAD_LINUX=keep
 ```
 
5. Pour ajouter un fond d'écran, il faut taper ces commandes : 
* `sudo apt-get install grub2-splashimages` qui va télécharger le paquet mentionné (c'est un paquet qui propose quleques images). Après cette installation, les images se trouvent dans __/usr/share/images/grub__
* `GRUB_BACKGROUNG="/usr/share/images/grub/xxx.tga"` qui affiche 809
 
  <p align="right">
 <img src="https://i.ibb.co/stZvP03/fond.png"/>
  </p 
  
6. La configuration d'un thème ne diffère pas vraiment de celle du fond d'écran. On doit : 
 * Installer le dépôt __grub2-themes-*__ avec la commande `sudo apt-get install grub2-themes-*`. Après cette installation, on se retrouve avec deux dossiers de thèmes _ubuntu-mate__ et __ubuntustudio__
 * Choisir un thème, __ubuntu-mate__ dans mon cas.
 * modifier la variable __GRUB_THEME__ dans le fichier `/etc/default/grub` en tapant cette ligne `GRUB_THEME=/boot/grub/themes/ubuntu-mate/theme.txt`
  
   <p align="right">
  <img src="https://i.ibb.co/xs5cKwf/theme.png"/>
   </p 
 
7. Pour ajouter une netrée permettant d'arrêter la machine et une autre permettant de la redémarrer, il suffit de modifier le script __40_custom__ se trouvant dans __/etc/grub.d/__.
 ``` bash
 menuentry 'Arrêt de la machine' {
 halt
 }
 menuentry 'Redémarrage de la machine' {
 reboot
 }
 ```

8. La configuration de GRUB pour que le clavier soit en français n'est pas fonctionnel sur __UBUNTU 21.10__

&#x1F4A1; __Notez bien__ que pour faire toutes ces modifications dans le fichier `/etc/default/grub`, on a utilise la commande `nano /etc/default/grub`.

  # Exercice 2 - Noyau
  __Objectif:__ Créer et installer un module pour le noyau
 
  __Démarche à suivre:__
  * Installation du paquet __build-essential__ qui va contenir tous les outils nécessaires (compilateurs, biblothèques) à la compilation de programmes en C (entre autres) avec la commande `sudo apt-get install build-essential`
  * Création d'un fichier __hello.c__ avec la commande `touch hello.c` et on y copie le code suivant : 
 
 ``` bash
#include <linux/module.h>
#include <linux/kernel.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("John Doe");
MODULE_DESCRIPTION("Module hello world");
MODULE_VERSION("Version 1.00");

int init_module(void)
{
   printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
   return 0;
}
 
void cleanup_module(void)
{
    printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
}
```
   * Création d'un fichier Makefile contenant :
 ``` C
1 obj-m += hello.o
2 
3 all:
4    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
5 clean:
6    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
7 install:
8   cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
 ```
 &#x274C; __ATTENTION__ Les lignes 4, 7 et 10 doivent commencer par une tabulation
 
  * Compilation du module à l'aide de la commande `make` puis son installation avec la commande `make install`
 La module est installé dans le dossier spécifié à la ligne 10
  * Chargement du module avec la commande `insmod hello.ko`, on peut vérifier dans le journal du noyau que le message "la focntion init_module() est appelée" a bien été inscrit.
  *Vérification que le module a bien été chargé avec la commande `lsmod`
 
 __Questions supplémentaires__
  * On décharge le module avec la commande `rmmod hello`, on peut vérifier dans le journal du noyau que le message "la focntion cleanup_module() est appelée" a bien été inscrit.
 
 &#x1F4A1; __Notez bien__ que la vérification du journal se fait avec la commande `dmesg` et qu'il faut utiliser la commande `modprobe -a` avnt la commande `insmod` .
 
 * Chargement automatique du module au démarrage du système : il faut inscrire le module dans le fichier __/etc/modules__ en rajoutant juste la ligne __hello_
  
  # Exercice 3 - Interception de signaux
  1. Création d'un script qui permet de recopier dans un fichier __tmp.txt__ chaque ligne saisie au clavier par l'utilisateur
 
 ``` bash
 #!bin/bash
 
 while (true); do
 read a
 echo $a >> tmp.txt
 done
 
  ```
  2. On lance le script et on appuye sur __CTRL+Z__ . Cette commande suspend le programme et pour qu'il suive son exécution, on tape la commande `fg`
 
 3. L'appui sur __CTRL+C__ arrête le programme.
 
 4. On va modifier le script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP (= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher “Impossible de me placer en arrière-plan”, et dans le second cas, il doit afficher “OK, je fais un peu de ménage avant” avant de supprimer le fichier temporaire et terminer le script.
 
 ``` bash
 #!bin/bash
 
 trap 'echo impossible de me placer en arrière-man' SIGTSTP
 trap 'echo OK, je fais un peu de méange avant ; exit' SIGINT
 trap 'rm tmpt.txt' EXIT
 
 while (true); do
 read a
 echo $a >> tmp.txt
 done
 ```
 5. La commande `KILL` arrête le programme.
 6. L'erreur est dû au fait que le fichier n'existe pas au début, il faut rajouter une condition de test pour rentrer à la boucle while dans le script bash.
 
  # Exercice 4 - Surveillance de l'activité du système
 
 1. La commande w affiche toutes les sessions ouverts avec le compte utilisateur actuel 
 
   <p align="right">
     <img src="https://i.ibb.co/vVVwzSz/Captur.png"/>
   </p  
    
    ==> image capturée après la commande `htop`
    
 
 2. Pour afficher l'istoriue des dernières connexions à la machine, on utilise la commande `last`
 3. Pour obtenir la version de noyau, il suffit de taper la commande suivante `uname -r`
 4. Pour récupérer toutes les informations sur le processeur au format JSON, on tape la commande `lshw -C CPU -json`
 5. Pour obtenir la liste des derniers démarrages de la machine, on tape la commande `journalctl --list-boots` et pour aﬀicher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot, on tape la commande `journalctl --bot=ID de lavant dernier boot qu'on récupère de la commande précedente`
 6. Pour faire en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message à l’écran d’une maintenance le 26 mars à minuit, il suffit de modifier le fichier __/etc/motd.tail__
  
 
 

  
  

