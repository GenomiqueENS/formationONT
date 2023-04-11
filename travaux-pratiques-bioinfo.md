<a href="https://genomique.biologie.ens.fr/"><img src="https://github.com/GenomiqueENS/eoulsan/raw/master/src/site/resources/images/logo-GenomiqueENS-90pxh.png" align="left"></a>
# Formation MinION Session Bioinformatique<br/>


Formateurs : Laurent Jourdren (*jourdren@bio.ens.psl.eu*) et Sophie Lemoine (*slemoine@bio.ens.psl.eu*)

Contact Plateforme GenomiqueENS :

* Site Web [https://genomique.biologie.ens.fr](https://genomique.biologie.ens.fr/)
* Courriel [genomique@bio.ens.psl.eu](mailto:genomique@bio.ens.psl.eu)
* Twitter [@Genomique_ENS](https://twitter.com/Genomique_ENS)

## Sommaire

* Présentation de la Plateforme Génomique de l'ENS
* Rappel du principe de la technologie ONT
* Présentation des différents types de flowcells et séquenceur ONT
* Principe de l’appel de base
* [Dépannage : Mise en service d’un MinION Mk1C](#config)
* [TP 1 : Connexion au MinION Mk1C en ligne de commande](#connexion)
* [TP 2 : Connexion à distance grace à MinKNOW Stand Alone GUI](#minknow-stand-alone-gui)
* [TP 3 : Transfert des données](#transfert)
* [TP 4 : Interface de MinKNOW, lancement d'un run et du basecalling via l'interface](#minknow)
* [TP 5 : Les fichiers bruts Fast5](#fast5)
* [TP 6 : Appel de Base en ligne de commande](#basecalling-cmdline)
* [TP 7 : Contrôle Qualité post run](#qc)
* Alignement
* Conclusion
* [Bibliographie](#biblio)


#### Informations utiles

Lors de ce TP, nous utilisons deux séquenceurs MinION et un séquenceur PromethION P2 solo:

* Un MinION Mk1B couplé à un PC sous Linux
* Un MinION Mk1C son nom de domaine est `minion01.example.com` ou `minion01`
* Un PromethION P2 couplé à un PC sous Linux

MinKNOW est le logiciel pilotant les séquenceurs MinION, GridION et PromethION.
Au cours de ce TP, la version de MinKNOW utilisé est la 21.12.x datant du 01 février 2023 pour les séquenceurs MinION et la 22.07.5 pour le PromethION P2 solo.
Le séquenceur P2 solo nécessite actuellement une version différente de MinKNOW de celle requise pour piloter un MinION Mk1B.
Il est fortement déconseillée d'utiliser la version de MinKNOW pour le MinION Mk1B avec le P2 solo.
À terme, d'ici quelques mois les deux versions seront fusionnées.

Les données utilisées lors de ce TP sont celles qui ont été produites lors de la session expérimentale de cette formation le 15 mars 2021. Le run effectué lors de la session expérimentale de la formation se nommait *TOTO*, un peu plus 21 000 lectures avaient été produites en utilisant une flowcell de type *FLO-MIN106* et le kit *SQK-PBK004*.
Deux échantillons avait été multiplexé.

**Note :** Les mots de passes utilisés lors de ce TP sont ceux utilisés par défaut par ONT. Il convient évidemment de les changer lors de mise en production d’un séquenceur.


<a name="config"></a>
## Dépannage : Mise en service d’un MinION Mk1C

La version du système préinstallé sur les MinION Mk1C est (était ?) notoirement boguée notamment en ce qui concerne la configuration réseau via MinKNOW (Elle ne fonctionne pas). Cette configuration initiale est certes délicate à mettre en place mais une fois les mises à jour du système effectuées, l’environnement logiciel du séquenceur s’avère stable.

Dans cette partie, vous trouverez la procédure à suivre pour mettre en service un MinION Mk1C.

Pour réaliser cette opération, il est nécessaire de disposer :

* Un câble réseau
* Un ordinateur avec une carte Wifi sous Linux, macOS ou Windows 10 (version supérieure ou égale à celle de septembre 2017)
* Un séquenceur MinION Mk1C

La procédure à suivre est la suivante :

1. Branchement de l’appareil au réseau électrique. Attention, les prises électriques mâles des appareils ONT sont parfois capricieuses, une multiprise peut être nécessaire pour brancher correctement l’appareil
2. On allume l’appareil
3. À l’aide d’ordinateur, connectez-vous en Wifi au Hotspot *MC-XXXXXX* créé par le séquenceur. Le mot de passe de ce Hotspot est *WarmButterflyWings98*
5. Trouvez l'adresse IP de la passerelle du réseau à l'aide de la commande `ipconfig` (Windows), `route -n get default` (macOS) ou `ip route` (Linux). Celle-ci est également l'adresse du MinION Mk1C sur ce réseau
4. Connectez-vous maintenant en SSH au séquenceur au compte d'administration *minit* (dans l'exemple ci-dessous, 10.42.0.1 correspond à l'adresse IP l'adresse IP de la passerelle du réseau récupérée à l'étape précédente; le mot de passe d'origine du compte d'administration est *minit*)
```bash
ssh minit@10.42.0.1
```

Une fois que nous avons accès au système du séquenceur, nous allons pouvoir agir sur la configuration réseau.
Pour cela, il faut utiliser la commande `nmcli` de l’utilitaire *NetWork Manager* qui permet de gérer la configuration réseau de l’ensemble du système.
Dans un premier temps, nous pouvons :

* Lister les configurations réseaux (les configurations actives apparaissent en vert)
```bash
nmcli connection show
```
* Afficher les caractéristiques d’une configuration réseau
```bash
nmcli connection show static
```
* Afficher les caractéristiques de la connexion filaire actuelle (notamment son adresse MAC)
```bash
nmcli device show eth0
```

Il faut maintenant choisir la manière dont vous allez configurer la connexion Ethernet filaire.
En mode dynamique (DHCP), ce qui est recommandé, le séquenceur va se connecter à un serveur DHCP pour obtenir sa configuration réseau.
Pour utiliser ce mode de fonctionnement, il sera peut-être nécessaire de fournir l'adresse MAC de la carte réseau à votre service informatique pour qu'il autorise le séquenceur à recevoir sa configuration réseau.
La commande `sudo` permet de passer en mode administrateur le temps d’exécuter une commande.

* Option 1 : configuration dynamique (DHCP) de la connexion Ethernet.
```bash
sudo nmcli connection down static    # Désactive la configuration statique de la connexion fillaire
sudo nmcli connection up dhcp        # Active la configuration dynamique de la connexion fillaire
```
* Option 2 : configuration statique de la connexion Ethernet. Il est prudent de recopier la configuration avant toute modification des caractéristiques d’une connexion
```bash
sudo nmcli connection edit static    # Édite la configuration statique de la connexion fillaire
sudo nmcli connection down dhcp      # Désactive la configuration dynamique de la connexion fillaire
sudo nmcli connection up static      # Active la configuration statique de la connexion fillaire
```

Maintenant que la configuration a été correctement définie, il ne reste plus qu’à mettre à jour le système à l’aide des commandes suivantes :
```bash
sudo apt update                      # Mise à jour de la liste des paquets disponibles 
apt list --upgradable                # Affiche les paquets pour lesquels une mise à jour est disponible
sudo apt dist-upgrade                # Lance la mise à jour du système
sudo shutdown --reboot now           # Redémarre le système
```



<a name="connexion"></a>
## TP 1 : Connexion au MinION Mk1C en ligne de commande

Le MinION Mk1C comme tous les séquenceurs d’ONT couplés à un ordinateur, fonctionne sous un système Linux.
Oxford Nanopore Technologies, laisse à ses utilisateurs un accès total au système d’exploitation de ses machines au travers de connexions de type SSH.
SSH (Secure Shell) est à la fois un programme informatique et un protocole de communication sécurisé permettant l’accès à distance à des ordinateurs via la ligne de commande.

Dans ce TP, nous verrons comment se connecter à un MinION Mk1C via la commande `ssh` et nous récupérerons quelques informations sur le système informatique pilotant le séquenceur.

* Ouvrir l’application `Terminal` de macOS (disponible dans le dossier *Applications/Utilitaires*) ou Linux (Appuyer sur la touche Windows du clavier puis taper `Terminal`) et se placer dans le dossier *Formation-MinION* sur le Bureau
```bash
cd ~/Desktop/Formation-MinION
```

* Connectez-vous au MinION en ligne de commande (le mot de passe est "**minit**")
```bash
ssh minit@minion01
```

**Question 1 : Sous quel système d’exploitation tourne le MinION Mk1C ?**

**Question 2 : Quel type de processeur (CPU) utilise le MinION Mk1C ? Quelle conséquence cela a-t-il ?**

**Question 3 : Combien de paquets du système peuvent-ils être mis à jour ?**

* Une fois connecté sur le MinION, nous disposons d’un contrôle complet sur le fonctionnement de l’ordinateur pilotant le séquenceur. On peut ainsi voir en temps réel les programmes en cours d’exécution sur à l’aide de la commande `htop` :
```
htop
```

**Question 4 : Combien de cœur possède le processeur (CPU) ?**

**Exercice 5 : Rechercher MinKNOW dans la liste des processus. On s’aperçoit que MinKNOW que MinKNOW est composé d’au moins deux parties (le cœur du logiciel et l’interface graphique)**

**Question 6 : Quel est le processus qui utilise le plus le processeur ? À quoi sert-il ?**

* Appuyez sur la touche **q** pour quitter le programme `htop`

* Les données de séquençages sont stockées dans la partition `/data`, déplacez-vous-y pour y lister les derniers runs :
```bash
cd /data
ls -l
```

**Question 7 : Quels sont les dossiers correspondants à des runs ?**

* L’espace disponible sur cette partition est accessible à l’aide de la commande `df`:
```bash
df -h /data
```

**Question 8 : Combien d’espace disponible reste-t-il pour stocker de nouveaux runs ?**

**Avertissement :** Sur le MinION Mk1C, en mode ligne de commande, vous pouvez faire ce que vous voulez.
Il convient donc d’être __extrêmement__ prudent, car vous n’aurez pas de message d’avertissements (comme lorsque vous souhaitez supprimer des données avec la commande `rm`).


<a name="minknow-stand-alone-gui"></a>
## TP 2 : Connexion au Mk1C à distance gràce à MinKNOW Stand Alone GUI

L’écran du MinION est relativement petit et pas toujours très pratique à utiliser, c’est pour cette raison (et aussi, car le MinIT ne disposait pas d’écran) que la société ONT a développé le logiciel *MinKNOW Stand Alone GUI* qui permet de contrôler à distance un ou plusieurs séquenceurs.

Dans ce TP, nous verrons comment installer ce logiciel sur un ordinateur de bureau (un iMac) et le configurer pour prendre le contrôle d’un séquenceur.

L’application *MinKNOW Stand Alone GUI* est disponible sur différents supports :
* [Windows et macOS](https://community.nanoporetech.com/downloads)
* [Android](https://play.google.com/store/apps/details?id=com.nanoporetech.minknowui)
* [IPhone et iPad](https://apps.apple.com/fr/app/minknow/id1504645283)

Pour le moment, il n’existe de pas de version pour les systèmes Linux.

* Installation du MinKNOW Stand Alone GUI
    * Allez dans le dossier *Outils* des documents de la formation présent sur le bureau de l’ordinateur
    * Faire un double clic sur *MinKNOW-UI-OSX-5.4.5-intel.dmg*
    * Dans la fenêtre qui s’ouvre, déplacer l’icone *MinKNOW UI* dans le raccourci vers le dossier *Applications*

* Configuration de l’application
    * Au démarrage MinKNOW vous propose de vous connecter avec votre compte Nanopore. Choisissez *Log with your Nanopore account*, et saisissez vos identifiants
    * Vous arrivez sur la page du *Connection manager*. Le mode tutoriel est activé, quittez le en cliquant sur le bouton **⋮**
    * Ajoutez une connexion à un séquenceur en cliquant sur le bouton **⊕ Add host**, et rentrez le nom de domaine du séquenceur `minion01`
    * Dans la section *Saved Host*, devrait apparaitre une icone *Mk1C* selon le sequençeur auquel vous vous êtes connecté
    * Cliquez sur l’hôte crée pour pouvoir controler à distance le sequenceur

**Note :** Sous macOS, les données de l’application `MinKNOW UI` sont stockées dans le dossier *~/Library/Application Support/MinKNOW* (*~* correspond au chemin de votre dossier personnel).
Pour réinitialiser l'application, il suffit de supprimer ce dossier et de relancer l’application.


<a name="transfert"></a>
## TP 3 : Transfert des données présentes sur le Mk1C

Les séquenceurs MinION Mk1C, GridION et PromethION enregistrent par défaut (et cela est fortement conseillé) les données produites lors du séquençage dans le stockage interne de l’appareil (les unités de stockage ont des caractéristiques compatibles avec le débit du séquenceur).
Il est donc nécessaire de pouvoir transférer des données depuis et vers le séquenceur.
Il existe de très nombreuses méthodes pour transférer des données de et vers un MinION Mk1C :

* Disque dur ou clé USB
* micro SD-Card
* Partage SMB
* Partage NFS
* Tout autre méthode de partage réseau que permet Linux en ligne de commande (NFS, SSH, SFTP, FTP…)

Dans ce TP, nous allons nous concentrer dans ce TP sur 4 méthodes, trois en utilisant MinKNOW (Clé USB, Partage SMB et montage réseau) et une en ligne de commande (SFTP).

### Support Physique

Vous pouvez insérer un disque dur, une clé USB ou une carte SD dans le MinION ainsi réaliser des transferts de fichiers via l'interface graphique de MinKNOW.

* Il vaut mieux éviter de se servir de ces supports physiques pour y écrire les données produites au cours du run (idem pour les montages réseaux) car cela pourrait faire un goulot d’étranglement et avoir pour conséquence une perte de données (Nous ignorons s’il y a un cache pour les données en cours d’acquisition).
* Le choix type de système de fichier du support physique est très important car :
    * les systèmes de fichiers natifs de macOS (HFS+, AFS) ne sont pas supportés
    * les systèmes de fichiers FAT historiques (FAT, FAT32, vFAT) ne supportent pas des fichiers de plus de 4 Go
    * Il est préférable d’utiliser exFat (supportant les fichiers ⩾ 4 Go et disponible en lecture/écriture sous Windows, macOS et Linux) ou Ext2/3/4 (Linux).

* Ouvrez *MinKNOW*, allez dans *Host settings* / *File Manager* / Onglet *Internal*

**Exercice 1 : Brancher une clé USB et à l’aide de l’interface graphique copier un fichier de la clé dans le dossier */data*.**

**Exercice 2 : À l’aide de l’interface graphique copier un petit dossier vers la clé USB. Démontez-la et retirer là et branchez-la sur l’ordinateur pour vérifier que vous pouvez accéder aux données.**

### Partage SMB

Dans cette partie, nous allons voir comment accéder depuis l’ordinateur aux fichiers présents sur le MinION à l’aide d’un montage SMB.
Le protocole SMB (Server Message Block) est un protocole permettant le partage de ressources (fichiers et imprimantes) sur des réseaux locaux à l’origine avec des PC sous Windows.
Désormais ce protocole est pris en charge par macOS et Linux.

* Création du montage
    * Dans *MinKNOW*, allez dans *Host settings* / Section *Device settings* / Partie *Disk management*
    * Activer "l'interupteur" *Share* à coté de la partion */data*
    * Une boite de dialogue apparaît vous demandant de choisir un mot de passe pour ce partage. Remplissez-la.
    * Le partage est alors activé

* Montage du partage sur l’ordinateur
    * Dans le Finder et dans le menu sélectionnez *Aller* / *Se connecter au serveur…*
    * Utiliser *smb://minion01*. Appuyer sur le bouton *Se connecter*
    * Une boite de dialogue apparaît. Choisissez *Utilisateur référencé*, le nom de l’utilisateur est **minit** et le mot de passe celui que vous avez précédemment choisi. Appuyez sur le bouton *Se connecter*

**Exercice 3 : Utiliser ce partage pour copier des fichiers vers le montage. Vérifiez que les fichiers copiés sont bien visibles dans l’interface de MinKNOW**

### Montage d'un disque réseau sur le MinION

Dans la méthode précedente, nous rendions accessible les données du MinION sur un autre ordinateur.
Il est également possible de faire l'inverse, c'est à dire monter un disque réseau sur le MinION.

Pour cela il faut procéder de la manière suivante :

* Dans *MinKNOW*, allez dans *Host settings* / *File Manager* / Onglet *Internal*
* Appuyer sur le Bouton *Add a network drive*

Deux types de protocoles de montage de disque réseau sont disponibles :

* SMB, protocole le plus courant dans le monde Windows et macOS généralement plus simple à mettre en oeuvre
* NFS, protocole venant du monde Unix/Linux

Le meilleur moyen d'utiliser cette possibilité est d'utiliser l'infrastrure de stockage de votre établissement ou de disposer d'un [NAS (Network Attached Storage)](https://fr.wikipedia.org/wiki/Serveur_de_stockage_en_r%C3%A9seau), qui permettra de vous connecter avec l'un ou l'autre des protocoles.

La [documentation de MinKNOW](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/minion-mk1c-user-manual/v/mkc_2005_v1_revt_27nov2019/mount-network-drive) explique comment configurer votre montage réseau en fonction du protocole retenu.

### SFTP

SFTP est l’évolution sécurisée du protocole de transfert de fichier FTP qui date des années 70.
SFTP est habituellement utilisable sur une machine dès qu’un serveur SSH est configuré, comme c’est le cas sur MinION Mk1C.

* Récupération d’un run. Ouvrir l’application `Terminal` de macOS et lancer les commandes suivantes :
```bash
cd ~/Desktop/Formation-MinION/Données
sftp -r minit@minion01:/data/TOTO .
```

* L’option `-r` de `sftp` active une copie récursive des dossiers/fichiers.

* Dépot de données sur le MinION (ex: pour relancer l’appel de base d'un run) :
```bash
cd ~/Desktop
sftp -rp TOTO minit@minion01:/data/TOTO-copie .
```

* L’option `-p` de `sftp` permet de préserver les permissions et les dates des fichiers.


<a name="minknow"></a>
## TP 4 : Interface de MinKNOW, lancement d'un run et du basecalling via l'interface

Dans ce TP nous allons prendre en main MinKNOW, l'interface graphique permettant le contrôle du MinION Mk1b, du Mk1C et du PromethION P2 Solo.
Même si les versions de MinKNOW varient pour le moment selon le séquenceur à piloter, le paramétrage des runs sont similaires voire identiques.
Cette interface permet l'accès à un certain nombre de paramètres tels que les lancement des runs, du basecalling, de l'alignements des lectures obtenues, etc…


### Utilisation des MinION et du PromethION via leurs interfaces graphiques

Après lancement de l'application vous avez accès aux séquenceurs qui lui sont accessibles.
Choisissez le séquenceur sur lequel vous travaillez. Placez une flowcell dans l'emplacement prévu.
Vous devez maintenant voir la flowcell que vous avez mis en place sur l'interface.

Le menu accessible sur la gauche de l'application vous propose 5 options : 
- Start
- Sequencing overview
- Experiments
- System messages
- Host settings

Parcourez les Host settings.

**Exercice 1 : Dans quel sous menu des settings devez vous aller pour redémarrer ou éteindre le système d'exploitation du séquenceur ?
Où verifier que MinKNOW est bien à jour ?**

Ce sous menu vous permet de :

- connaitre l’espace qu’il vous reste sur vos disques
- naviguer dans vos résultats, run par run
- effectuer la mise à jour du système d’exploitation de votre appareil
- mettre à jour MinKNOW


### Vérification initiale du séquenceur ou de la flowcell

A la réception du séquenceur vous devez verifier son état.
Pour le faire, vous trouverez une flowcell factice en plastique blanc dans la boite de l'appareil.
Il s’agit de la flowcell de configuration (CTC). Insérez là dans l'emplacement de la flowcell et cliquez sur start.
En choisissant la section Hardware check dans la section *Start*, vois pouvez lancer la vérification de votre matériel.

**Exercice 2 : Lancez le Hardware Check**

Avant chaque lancement de run, vous devez aussi vous assurer que la flowcell rempli les conditions d’utilisation: il est nécessaire de vérifier le nombre de pores disponibles sur la flowcell avant de charger les échantillons.
Le nombre de pores disponibles doit être supérieur à :

- 50 dans le cas d’une flowcell Flongle
- 800 dans le cas d’une flowcell MinION
- 5000 dans le cas d’une flowcell PromethION

Les flowcells sont remplacées si ce n’est pas le cas !

**Exercice 3 : Revenez au menu précédent et lancez le Flowcell Check**

### Lancement d'un run

**Exercice 4 : Paramétrez et lancez votre run**

Il faut définir :

- Votre expérience
- Le kit utilisé
- Le type de basecalling (choix du modèle de réseau de neurone) s’il est fait à la volée
- Les format de sortie de vos données
- L’alignement à la volée ou non et par conséquent, les séquences références

Commençons !


**Définissez votre expérience et passez à la selection du kit (N’oubliez pas de lui donnez un nom !) :**

Dans la section permettant le choix du kit de séquençage à utiliser, tous les kits sont disponibles.
Il est possible de les filtrer selon ce que l’on séquence, selon les banques faites…
Choisissez ce qui vous intéresse.
Sur notre plateforme nous utilisons le kit SQK-PBK004, C’est un kit ADN avec PCR.
Il est important de ne pas se tromper: chaque kit possède des spécificités d’amorces et cet aspect sera primordial pour la partie basecalling, demultiplexing…
Attention, la chimie est en ce moment en cours d'évolution et les kits ne sont pas tous disponibles selon les Flowcells et les séquenceurs.


**Passez au choix des options de runs :**

Selon le type de séquençage que vous souhaitez faire, votre run va durer plus ou moins longtemps.
Pour un RNASeq, un run de 72h est adapté. Si vous souhaitez tester la presence ou non d’une bactérie, 20 minutes peuvent suffire (votre flowcell peut être utilisée plusieurs fois). Le voltage initial de la flowcell peut être modifié mais il vaut mieux être expert pour cela.
Le contrôle actif des canaux est enclenché ce qui autorise MinKNOW a monitorer les canaux en permanence pour une meilleure performance de ceux-ci.
Le temps entre chaque changement des canaux (mux scan) est aussi paramétrable. Vous pouvez également sauvegarder un pourcentage de pores pour les faire intervenir dans la durée du run.
C'est dans cette section que vous pouvez parametrer MinKNOW pour qu'il fasse de l'adaptive sampling en enrichissant ou en rejetant certaines séquences. 
Dans ce cas vous devez fournir une séquence FASTA de référence (type génome) et un fichier BED de ce que vous voulez enrichir ou rejeter.
Vous pouvez également spécifier des code-barres à enrichir.
Il faut noter que ces deux possibilités, adaptive sampling ou barcode balancing, sont en version beta.
Vous pouvez jouer avec ces options pendant le TP.


**Passez à la configuration du basecalling :**

L’appel de base peut être réalisé à la volée ou après le run.
Il peut être réalisé sur le Mk1C ou un ordinateur indépendant.
Nous allons voir comment le lancer à la volée. 
Les paramètres importants restent les mêmes quelque soit la machine choisie pour réaliser l’appel de base.

Deux modes de basecalling sont possibles :

- Fast (fast) : Pratique pour le diagnostique parce rapide
- High-accuracy (hac) : Plus long mais moins d’erreur

**Note :** Un mode *super acurracy (sup)* existe mais il est seulement disponible en ligne de commande.
Il est indispensable de disposer d'une carte GPU puissante pour réaliser l'appel de base dans ce mode.

**Passons aux code-barres :**

Dans le  cas d’utilisation de code-barres, vous pouvez jouer sur plusieurs paramètres :

- Suppression des code-barres aux extrémités des données basecallées (*Trim barcodes*)
- Recherche des code-barres à chaque extrémité de la lecture pour classifier la lecture : si un seul des code-barres est trouvé, la lecture est perdue (*Barcode both ends*)
- Recherche de code-barre au milieu de la lecture: Elimination de la lecture si un code barre est trouvé (*Mid-read barcode filtering*)
- Filtrage des code-barres selon leur score de façon à etre plus stingent sur leur qualité

**Attention :** le sequençage nanopore est encore imprecis.
Les sequences si elles sont petites comme des code-barres et qu'elles contiennent des erreurs peuvent être mal reconnues.
Vous risquez de perdre beaucoup à être trop stringent.


**Lancement de l'alignement à la volée :**

MinKNOW  peut lancer l’alignement à la volée. Minimap2 est le mapper qui est utilisé de façon standard.
Si vous souhaitez le faire, vous devez fournir un fichier FASTA de référence.
Si vous faites du RNA-seq, vous pouvez également donner en entrée de minimap2, un fichier BED12 définissant les jonctions de vos isoformes.
Vous pouvez utiliser paftools, un outil intégrer à minimap2, pour construire les BED12 correspondant à votre problématique à partir des fichiers d’annotation GTF qui sont plus courant.


**Quels sont les fichiers de sorties à choisir en sortie de MinKNOW ?**

- Des Fast5 : Ce sont les données brutes. Il est important de les conserver si l’on veut relancer le basecalling en fonction des évolutions de Guppy
- Des POD5 : Ce sont les données brutes. Il s'agit d'un nouveau format permettant une meilleure compression des données brutes.
- Des FASTQ : Ce sont les données basecallées, demultiplexées (si besoin) et classées en pass/fail
- Des BAM : Ce sont les données alignées si l’alignement à la volée a été demandé

Vous pouvez choisir le critère qui classera la lecture en pass ou fail.
Classiquement, les lectures aillant un score de qualité inférieur à 8 en mode "fast" sont considérées comme mauvaises (fail).
La valeur par défaut de ce seuil change selon le type d'appel de base (fast : 8, hac : 9 et sup : 10).
Les lectures peuvent être filtrées sur leur qscore minimal et/ou leur taille.
Vous pouvez aussi choisir de couper les lectures chimériques (voir section suivante) formées au moment du séquençage *Enable read splitting*.


**Quid du fichier Fast5 Bulk ?**

Dans ce type de fichier Fast5, MinKNOW ne fait pas de coupure entre chaque lecture d’un pore (*Advanced options*):

- elles restent liées en une longue séquence comprenant les adaptateurs et les sequences d'interet.
- il est possible de visualiser le signal et de voir les coupures déterminant les lectures dans BulkVis par exemple [Publi de Bulkvis].
Selon la séquence, il est possible que MinKnow ne coupe pas au bon endroit.
Des chimères peuvent être créees de cette façon.

Attention, cette option génère un gros volume de données.


<a name="fast5"></a>
## TP 5 : Les fichiers bruts Fast5

Les fichiers Fast5 sont les fichiers contenant les données brutes du séquençage.
Ces fichiers sont en fait des fichiers au format [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) qui utilise une structure arborescente propre à Nanopore.

Pour visualiser le contenu de ces fichiers, il est nécessaire d’utiliser des outils spécifiques. Dans le cadre de TP, nous utiliserons l’outil HDFView qui permet de visualiser des fichiers au format HDF5.

**Note 1:** Depuis la version 21.05.8 de MinKNOW (juin 2021), le format des fichiers Fast5 a évolué.
Le signal electrique est désormais compressé par défaut à l'aide de l'algorithme [VBZ](https://github.com/nanoporetech/vbz_compression) developpé par ONT.
Cet algorithme permet de reduire d'environ 1/3 la taille des données précédement compressées à l'aide de l'algorithme gzip et de réduire très fortement les temps de compresssion/décompression.
Cette algorithme n'étant pas standard, le signal electrique n’est plus visualisable dans HDFView pour les données compressées avec VBZ.

**Note 2:** Le format Fast5 ne sera plus le format de fichier de sortie par défaut d'ici quelques mois.
Le format POD5 prendra la relève, ce format est déjà disponible dans les options de MinKNOW.

Dans ce TP, nous allons explorer le contenu de ces fichiers Fast5 afin de mieux comprendre le principe le l’appel de base que nous aborderons dans le prochain TP.

* Installation de HDFView sous macOS
    * Aller dans le dossier *Outils* des documents de la formation présent sur le bureau de l’ordinateur
    * Faire un double clic sur *HDFView-3.1.2.dmg*
    * Acceptez les conditions d’utilisation
    * Une fenêtre avec une icone *HDFView* apparait, cliquez sur cette icône pour lancer l’application

* Installation de HDFView sous Linux
**TODO**

Nous allons maintenant ouvrir un des fichiers Fast5 pour en visualiser le contenu

* Allez dans le menu *File* et sélectionnez *Open*
* Appuyez sur le bouton *Option* et sélectionnez *All Files*
* Sélectionnez un des fichiers Fast5 présent dans le dossier *Données* des documents de la formation présent sur le bureau de l’ordinateur.

**Question 1 : Combien y a-t-il d’éléments dans le panneau de gauche et pourquoi ?**

* Sélectionnez un des éléments du fichier et allez dans le sous-dossier *Raw* et double-cliquez sur le « fichier » *Signal*
* Une fenêtre apparaît avec un tableau contenant une seule colonne. Il s’agit des valeurs brutes du séquençage
* Sélectionnez la première et seule colonne et appuyez ensuite le bouton en haut à gauche pour visualiser le signal. Une fenêtre avec les options de visualisation apparaît, appuyez simplement sur *OK*

**Question 2 : Identifiez la zone du signal correspondant à la queue polyA de la lecture (toutes les lectures n’en possèdent pas)**

**Exercice 3 : Dans les sous dossiers d’une lecture, retrouvez le numéro de pore, le numéro de la flowcell et la date de début de run**


<a name="basecalling-cmdline"></a>
## TP 6 : Appel de Base en ligne de commande

L’appel de base est réalisé à l’aide du logiciel Guppy développé par Oxford Nanopore Technologies.
D’autres outils existent/existaient/existeront (Guppy va être remplacé par Dorado d’ici quelques mois) pour cette tâche, mais il est recommandé d’utiliser celui d’ONT qui certainement aujourd’hui le plus performant.
Oxford Nanopore propose également d’autres logiciels pour l’appel de base mais ceux-ci sont dédiés à la recherche algorithmique et il ne vaut mieux pas les utiliser en production.

Dans ce TP, nous verrons comment lancer l’appel de base en ligne de commande et nous explorerons les fichiers générés lors de ce traitement.

Pour fonctionner, il est nécessaire de fournir à Guppy un fichier de configuration décrivant le modèle à utiliser pour effectuer l’appel de base.
Celui-ci peut être automatiquement déterminé par Guppy si on lui fournit à l’aide des arguments `--flowcell` et `--kit`.
Cependant si cette solution est choisie le mode *haute précision (hac)* sera automatiquement sélectionné.
Dans le cadre des données utilisées pour ce TP, ce sera la configuration *dna_r9.4.1_450bps_hac*. Afin de réduire les temps de calcul, nous forcerons l’utilisation de la configuration *dna_r9.4.1_450bps_fast* qui permettra d’effectuer l’appel de base en mode rapide.

**Note :** Si vous disposez d'une carte GPU puissante (non recommandé sur un MinION Mk1C), vous pouvez également forcer l'utilisation du mode super accuracy à l'aide de la configuration *dna_r9.4.1_450bps_sup*.
La qualité de l'appel de base en mode super accuracy est légèrement supérieure au mode high accuracy mais demande beaucoup plus de temps de calcul que ce dernier.
Le mode high accuracy est généralement un bon compromis entre temps de calcul et qualité de l'appel de base.

* Ouvrez le logiciel `Terminal` sur l'ordinateur et connectez-vous via `ssh` au séquenceur

* Pour connaître les fichiers de configuration qui seront automatiquement sélectionnés en fonction de la flowcell et du kit, il suffit de lancer la commande suivante :
```bash
guppy_basecaller --print_workflows
```

* L’appel de base peut-être lancer en ligne de commande de la manière suivante sur le MinION Mk1C (le dossier de sortie doit exister):
```bash
mkdir /data/appel_de_base_ligne_de_commande_guppy_server
/usr/bin/guppy_basecall_client --port 5555 \
                               --server_file_load_timeout 600 \
                               --num_callers=1 \
                               --input_path /data/TOTO/no_sample/20210315_1508_MC-110337_0_FAO31058_dad08772/fast5 \
                               --fast5_out \
                               --save_path /data/appel_de_base_ligne_de_commande_guppy_server \
                               --compress_fastq \
                               --recursive \
                               --min_score 40.000000 \
                               --config dna_r9.4.1_450bps_fast.cfg \
                               --barcode_kits SQK-PBK004
```

* À titre d’information, on peut également lancer Guppy hors mode serveur avec la ligne de commande suivante (il faut retirer l’option `--device` si vous ne disposez pas d’un GPU) :

```bash
mkdir /data/appel_de_base_ligne_de_commande_guppy
/usr/bin/guppy_basecaller --device auto \
                          --save_path /data/appel_de_base_ligne_de_commande_guppy \
                          --input_path /data/TOTO/no_sample/20210315_1508_MC-110337_0_FAO31058_dad08772/fast5 \
                          --fast5_out \
                          --compress_fastq \
                          --recursive \
                          --min_score 40.000000 \
                          --config dna_r9.4.1_450bps_fast.cfg \
                          --barcode_kits SQK-PBK004
```

* Remarquera que cette ligne de commande précédente est plus lente (14m30s) que la précédente (7m30) car les paramètres de parallélisation n’ont pas été optimisés.

* Pour plus d’informations, notamment sur comment configurer l’alignement et le rognage des adaptateurs, il convient de se reporter à la [documentation de Guppy](https://community.nanoporetech.com/protocols/Guppy-protocol/v/gpb_2003_v1_revu_14dec2018) et à l’aide en ligne de commande (`guppy_basecaller --help`).

**Question 1 : Quel est l’intérêt d’utiliser Guppy en mode serveur sur un MinION Mk1C ? et sur GridION ou PromethION ?**

**Question 2 : Quel est le désavantage d’utiliser Guppy en mode serveur ?**

* À la fin de l’appel de base on obtient l’arborescence suivante :
```
.
├── barcode01
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_0_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_1_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_2_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_3_0.fastq.gz
│   └── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_5_0.fastq.gz
├── barcode02
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_0_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_1_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_2_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_3_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_4_0.fastq.gz
│   └── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_5_0.fastq.gz
...
├── barcode12
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_0_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_1_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_2_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_3_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_4_0.fastq.gz
│   └── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_5_0.fastq.gz
├── guppy_basecall_client_log-2021-03-21_13-07-22.log
├── guppy_basecall_client_log-2021-03-21_13-14-23.log
├── sequencing_summary.txt
├── sequencing_telemetry.js
├── unclassified
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_0_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_1_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_2_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_3_0.fastq.gz
│   ├── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_4_0.fastq.gz
│   └── fastq_runid_ec55bcd2efa6d77e9b07d97d4dbbdf4ea224aadb_5_0.fastq.gz
└── workspace
    ├── FAO31058_ec55bcd2_0.fast5
    ├── FAO31058_ec55bcd2_1.fast5
    ├── FAO31058_ec55bcd2_2.fast5
    ├── FAO31058_ec55bcd2_3.fast5
    ├── FAO31058_ec55bcd2_4.fast5
    └── FAO31058_ec55bcd2_5.fast5
```

**Question 3 : Pourquoi avons-nous plusieurs fichiers FASTQ par code-barre ? Que vaut-il mieux faire avant de réaliser analyse secondaire ?**

**Question 4 : Deux échantillons ont été déposés sur la flowcell lors du run. Pourquoi avons-nous une 12 dossiers pour des codes-barres lieu de 2 ?**

**Question 5 : À quoi correspond le dossier les fichiers FASTQ du dossier *unclassified* ?**

**Exercice 6 : Ouvrez un des fichiers Fast5 produits lors du démultiplexage avec HDFView. Comparez leur structure avec celle des fichiers Fast5 avant démultiplexage. Retrouvez les séquences appelées au format FASTQ dans les fichiers Fast5.**

* Sur l’ordinateur, allez dans le dossier *Formation-MinION/appel_de_base* sur le Bureau et ouvrez avec Firefox le fichier *sequencing_telemetry.js*

**Question 7 : Que contient ce fichier ? Avons-nous vu déjà une partie de ces informations quelque part ?**

* Toujours dans le même dossier, ouvrez le fichier *sequencing_summary.txt* à l’aide d’un tableur (Microsoft Excel ou Libreoffice Calc), il s’agit d’un fichier texte tabulé (TSV).

**Question 8 : Que contient ce fichier ? Avons-nous vu un fichier avec un nom identique précédemment ? Quelles sont les différences entre ces fichiers ?**

**Question 9 : Trouvez les colonnes pour identifier les lectures passant les filtres qualité, la longueur des lectures, la qualité moyenne des lectures et le code barre identifié**


<a name="qc"></a>
## TP 7 Contrôle Qualité post run

Dans ce dernier TP, nous comparons les rapports de contrôle qualité produits par différent outils et nous en comparerons les avantages et les inconvénients.

**Note :** Pour des raisons de simplicité et de rapidité, nous utiliserons des rapports générés avant le début du TP. PycoQC et ToulligQC sont des outils qui s’installent très facilement et qui s’executent en quelques secondes/minutes.

### Rapport de MinKNOW

* À la fin d’un run, MinKNOW va sauver sous la forme d’un fichier PDF, les informations et les graphiques qu’il affichait au cours du run.
* Sur l’ordinateur, allez dans le dossier *Formation-MinION/qc/MinKNOW* sur le Bureau et ouvrez le rapport PDF.

**Question 1 : Que manque-t-il dans le rapport produit par MinKNOW à la fin du run ?**

### PycoQC

* PycoQC est un outil permettant de produire un rapport de contrôle qualité après démultiplexage. Il se base sur le contenu du fichier *sequencing_summary.txt*.

* PycoQC est un outil développé en Python. Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :
```bash
pip install pycoQC
```

* Avec les données que nous avons générées, la commande à lancer pour produire le rapport est la suivante :

```bash
pycoQC --summary_file sequencing_summary.txt \
       --html_outfile pycoQC-report.html \
       --json_outfile pycoQC-report.json
```

* Sur l’ordinateur, allez dans le dossier *Formation-MinION/qc/PycoQC* sur le Bureau et ouvrez le rapport HTML.

**Question 2 : Qu’apporte PycoQC par rapport produit par MinKNOW ?**

### ToulliqQC

* ToulliqQC est un autre outil permettant de produire un rapport de contrôle qualité après démultiplexage. Il est développé par la plateforme génomique de l’ENS. Il se base sur les contenus des fichiers *sequencing_summary.txt* et *sequencing_telemetry.js*.

* ToulliqQC est un outil développé en Python. Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :

```bash
pip install toulligqc
```

* Avec les données que nous avons générées, la commande à lancer pour produire le rapport est la suivante :

```bash
toulligqc --report-name Formation_MinION \
          --telemetry-source sequencing_telemetry.js \
          --sequencing-summary-source sequencing_summary.txt \
          --barcodes BC11,BC12 \
          --output .
```

* Sur l’ordinateur, allez dans le dossier *Formation-MinION/qc/ToulligQC* sur le Bureau et ouvrez le rapport HTML.

**Note :** Le rapport produit ici a été réalisé par la version beta 3 de ToulligQC 2.0. La version finale est attendue d’ici la fin du mois.

**Question 3 : Qu’apporte ToulligQC par rapport produit par PycoQC ?**

**Question 4 : Que manque-t-il à ToulligQC ?**

### Nanocomp

NanoComp est un outil faisant partie d'une suite appelée NanoPack. Il permet de comparer des echantillons d'un même run via le fichier sequencing_summary.txt, des fichiers fastq, des fasta, des alignements via des fichiers bam.
Il est donc utilisable à plein d'étapes de l'analyse se qui en fait un outil interessant. 
La suite est toujours maintenue.
NanoPack est un outil développé en Python. Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :

```bash
pip install nanopack
```

* Avec les données que nous avons générées, la commande à lancer pour produire le rapport est la suivante :


```bash
NanoComp --summary /data/sequencing_summary.txt \
 --barcoded \
 -o /res/NanoComp_summary-plots

```
* Sur l’ordinateur, allez dans le dossier *formation-minion/qc/NanoComp* sur le Bureau et ouvrez le rapport HTML.


<a name="biblio"></a>
## Bibliographie
* [Téléchargement des outils ONT](https://community.nanoporetech.com/downloads)
* [Notes de version de MinKNOW pour le MinION Mk1C](https://community.nanoporetech.com/downloads/minion-mk1c-software/release_notes)
* [Documentation de MinKNOW pour le MinION Mk1C](https://community.nanoporetech.com/protocols/minion-mk1c-protocol/v/mkc_2005_v1_revm_27nov2019/updating)
* [Documentation de Guppy](https://community.nanoporetech.com/protocols/Guppy-protocol/v/gpb_2003_v1_revu_14dec2018)
* [Documentation de MiniMap2](https://github.com/lh3/minimap2)
* [Documentation de PycoQC](https://github.com/tleonardi/pycoQC)
* [Documentation de ToulliqQC](https://github.com/GenomicParisCentre/toulligQC)
* [Documentation de Nanocomp](https://github.com/wdecoster/nanocomp)
* [Publi de Bulkvis](https://dx.doi.org/10.1093%2Fbioinformatics%2Fbty841)

