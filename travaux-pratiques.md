<a href="https://genomique.biologie.ens.fr/"><img src="https://www.outils.genomique.biologie.ens.fr/aozan/images/logo_genomicpariscentre-90pxh.png" align="left"> </a>
# Formation MinION Session Bioinformatique<br/>23 mars 2021


## Sommaire

* Présentation de la Plateforme Génomique de l'ENS
* Rappel du principe de la technologie ONT
* Présentation des différents types de flowcells et séquenceur ONT
* [Démo : Mise en service d’un MinION Mk1C](#config)
* [TP 1 : Connexion au MinION Mk1C en ligne de commande](#connexion)
* [TP 2 : Connexion à distance gràce à MinKNOW Stand Alone GUI](#minknow-stand-alone-gui)
* [TP 3 : Transfert des données](#transfert)
* [TP 4 : Interface de MinKNOW et lancement d’un run](#minknow)
* [TP 5 : Les fichiers bruts Fast5](#fast5)
* Principe de l’appel de base
* [TP 6 : Relancer un appel de base avec MinKNOW](#basecalling-minknow)
* [TP 7 : Appel de Base en ligne de commande](#basecalling-cmdline)
* [TP 8 : Contrôle Qualité post run](#qc)
* Alignement
* Conclusion
* [Bibliographie](#biblio)


#### Informations utiles

Lors de ce TP, nous utilisons deux séquenceurs MinION :

* Un MinION Mk1B couplé à un MinIT, son nom de domaine est `minion01.in-genomique.biologie.ens.fr`
* Et un MinION Mk1C son nom de domaine est `minion02.in-genomique.biologie.ens.fr`

Le MinION Mk1C peut être assimilé à un MinIT couplé à un MinION Mk1B doté d'un ecran tactile. Donc toutes les informations et procedures relatives au MinION Mk1C utilisées dans ce TP sont également valables pour le MinIT.

Les données utilisées lors de ce TP sont celles qui ont été produites lors de la précédente session expérimentale de cette formation.

**Note :** Les mots de passes utilisés lors de ce TP sont ceux utilisés par défaut par ONT. Il convient évidemment de les changer lors de mise en production d’un séquenceur.


<a name="config"></a>
## Démo : Mise en service d’un MinION Mk1C

La version du système préinstallé sur les MinION Mk1C est (etait ?) bogué. La configuration initiale du réseau est particulièrement délicate à mettre en place. Cependant une fois celle-ci mise en place et la mise à jour du système effectuée, l’environnement logiciel du séquenceur s’avère plutôt stable.

Dans cette partie, vous trouverez la procédure à suivre pour mettre en service un MinION Mk1C.

Pour réaliser cette opération, il est nécessaire de disposer :
* Un cable réseau
* Un ordinateur avec une carte Wifi sous Linux, macOS ou Windows 10 (version septembre 2017 minimum)
* Un séquenceur MinION Mk1C

La procédure à suivre est la suivante
1. Branchement de l’appareil au réseau électrique. Attention, les prises électriques mâles des appareils ONT sont parfois capricieuses, une multiprise peut être nécessaire pour brancher correctement l’appareil.
2. On allume l’appareil
3. À l’aide d’ordinateur doté une carte Wifi, connectez-vous au Hotspot *NOM_DU_HOTSPOT* créé par le séquenceur. Le mot de passe de ce Hotspot est *MOT_DE_PASSE*
4. Connectez-vous en SSH au séquenceur
```bash
ssh minit@192.168.0.1
```
5. Option 1 : configuration dynamique (DHCP) de la connexion Ethernet. Pour cela, on utilise la commande suivante :
```bash
sudo nmcli c down static
sudo nmcli c up dhcp
```
6. Option 2 : configuration statique de la connexion Ethernet. Pour cela, on utilise la commande suivante :
```bash
sudo nmcli c edit static
sudo nmcli c down dhcp
sudo nmcli c up static
```
7. Mise à jour du séquenceur
```bash
apt update
apt dist-upgrade
shutdown --reboot now
```


<a name="connexion"></a>
## TP 1 : Connexion au MinION Mk1C en ligne de commande

Le MinION Mk1C comme tous les séquenceurs d’ONT disposant d’une unité de calcul, fonctionne sous un système Linux. Oxford Nanopore Technologies, laisse à ses utilisateurs un accès total au système d’exploitation de ses machines au travers de connexions de type SSH. SSH (Secure Shell) est à la fois un programme informatique et un protocole de communication sécurisé permettant l'accès à distance à des ordinateurs en ligne de commande.

Dans ce TP, nous verrons comment se connecter via la commande `ssh` à un séquenceur et nous récupérerons quelques informations sur le système informatique pilotant le séquenceur.

* Ouvrir l’application `Terminal` de macOS (disponible dans le dossier *Applications/Utilitaires*) et se placer dans le dossier *formation-minion* sur le Bureau
```bash
cd ~/desktop/formation-minion
```

* Connectez-vous au MinION en ligne de commande (le mot de passe est "**minit**")
```bash
ssh minit@minion0X.in-genomique.biologie.ens.fr
```

**Question 1 : Sous quel système d’exploitation tourne le MinION Mk1C ?**

**Question 2 : Quel type de processeur (CPU) utilise le MinION Mk1C ? Quelle conséquence cela a-t-il ?**

**Question 3 : Combien de paquets du système peuvent-ils être mis à jour ?**

* Une fois connecté sur le MinION, nous disposons d’un contrôle complet sur le fonctionnement de l’ordinateur pilotant le séquenceur. On peut ainsi voir en temps réel les programmes en cours d’exécution sur à l’aide de la commande `htop` :
```
htop
```

**Question 4 : Combien de cœur possède le processeur (CPU) ?**

**Question 5 : Rechercher MinKNOW dans la liste des processus. On s’aperçoit que MinKNOW que MinKNOW est composé d’au moins deux parties (le cœur du logiciel et l’interface graphique)**

**Question 6 : Quel est le processus qui utilise le plus le processeur ? À quoi sert-il ?**

* On constate que le processus `guppy_basecall_server` consomme beaucoup de temps processeur. Appuyez sur la touche **q** pour quitter

* Les données de séquençages sont stockées dans la partition `/data`, deplacez-vous y pour y lister les derniers runs :
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

* Sur le MinION Mk1C, en mode ligne de commande, vous pouvez faire ce que vous voulez. Il convient donc d’être extrêmement prudent, car vous n’aurez pas de message d’avertissements (par exemple lorsque vous supprimer des données avec la commande `rm`).


<a name="minknow-standalone-gui"></a>
## TP 2 : Connexion à distance gràce à MinKNOW Stand Alone GUI

L’écran du MinION est relativement petit et pas toujours pratique, c’est pour cette raison (et aussi, car le MinIT ne disposait pas d’écran) que la société a développé le logiciel *MinKNOW Stand Alone GUI* qui permet de contrôler à distance un ou plusieurs séquenceurs.

Dans ce TP, nous verrons comment installer ce logiciel sur un ordinateur de bureau (un iMac) et le configurer pour prendre le contrôle d’un séquenceur.

L’application *MinKNOW Stand Alone GUI* est disponible sur différents supports :
* [Windows et macOS](https://community.nanoporetech.com/downloads)
* [Android](https://play.google.com/store/apps/details?id=com.nanoporetech.minknowui)
* [IPhone et iPad](https://apps.apple.com/fr/app/minknow/id1504645283)

Pour le moment, il n’existe de pas de version pour Linux.

* Installation du MinKNOW Stand Alone GUI
    * Allez dans le dossier *Outils* des documents de la formation présent sur le bureau de l’ordinateur
    * Faire un double clic sur *MinKNOW UI OSX-4.2.8.dmg*
    * Dans la fenêtre qui s’ouvre, déplacer l’icone *MinKNOW UI* dans le raccourci vers le dossier *Applications*

* Configuration de l’application
    * Au démarrage MinKNOW vous propose de vous connecter avec votre compte Nanopore. Choisissez *Continue as guest*
    * Vous arrivez sur la page du *Connection manager*. Le mode tutoriel est activé, quittez le en cliquant sur le bouton **⋮**
    * Ajoutez une connexion à un séquenceur en cliquant sur le bouton **⊕ Add  host**, et rentrez l’adresse IP ou le nom de domaine du séquenceur (voir les informations utiles au début de ce document)
    * Dans la section *Saved Host*, devrait apparaitre une icone *Mk1C* ou *MinIT* selon le sequençeur auquel vous vous êtes connecté
    * Cliquez sur l’hôte crée pour pouvoir controler à distance le sequenceur

**Note :** Sous macOS, les données de l’application `MinKNOW UI` sont stockées dans le dossier *~/Library/Application Support/MinKNOW* (*~* correspond au chemin de votre dossier personnel). Pour réinitialiser l'application, il suffit de supprimer ce dossier et de relancer l’application.


<a name="transfert"></a>
## TP 3 : Transfert des données

Les séquenceurs MinION Mk1C, GridION et PromethION enregistrent par défaut (et cela est fortement conseillé) les données produites lors du séquençage dans le stockage interne de l’appareil. Il est donc nécessaire de pouvoir transférer des données depuis et vers le séquenceur. Il existe de très nombreuses méthodes pour transférer des données de et vers un MinION Mk1C :
* Disque dur ou clé USB
* micro SD-Card
* Partage SMB
* Partage NFS
* Tout autre méthode de partage réseau que permet Linux en ligne de commande (NFS, SSH, SFTP, FTP…)

Dans ce TP, nous allons nous concentrer dans ce TP sur 3 méthodes, deux en utilisant MinKNOW (Clé USB et Partage SMB) et une en ligne de commande (SFTP).

### Support Physique

Vous pouvez insérer un disque dur, une clé USB ou une carte SD dans le MinION et réaliser des transferts de fichiers.

* Il vaut mieux éviter de se servir de ces supports physiques pour y écrire les données produites au cours du run (idem pour les montages réseaux) car cela pourrait faire un goulot d’étranglement et avoir pour conséquence une perte de données (Nous ignorons s’il y a un cache pour les données en cours d’acquisition).
* Le type de système de fichier du support physique est important :
    *  les systèmes de fichiers natifs de macOS (HFS+, AFS) ne sont pas supportés
    *  les systèmes de fichiers FAT historiques (FAT, FAT32, vFAT) ne supportent pas des fichiers de plus de 4 Go
    *  Il est préférable d’utiliser exFat (supportant les fichiers ⩾ 4 Go et disponible en lecture/écriture sous Windows, macOS et Linux) ou Ext2/3/4 (Linux).

* Dans *MinKNOW*, allez dans *Host settings* / *File Manager* / Onglet *Internal*

**Exercice 1 : Brancher une clé USB et à l’aide de l’interface graphique copier un fichier de la clé dans le dossier /data**

**Exercice 2 : À l’aide de l’interface graphique copier un petit dossier vers la clé USB. Démontez-la et retirer là et branchez-la sur l’ordinateur pour vérifier que vous pouvez accéder aux données.**

### Partage SMB

Dans cette partie, nous allons voir comment accéder depuis l’ordinateur aux fichiers présents sur le MinION à l’aide de SMB.

**Note :** Il faut noter qu’il est possible de faire l’inverse et de monter un partage réseau (SMB ou NFS) depuis l’interface de MinKNOW. Pour cela il faut aller dans l’onglet *Network* du *File Manager* de *MinKNOW*.

* Création du montage
    * Dans *MinKNOW*, allez dans *Host settings* / *File Manager* / Onglet *Internal*
    * Appuyer sur le Bouton grisé *Share Samba network* en haut à droite
    * Une boite de dialogue apparaît vous demandant de choisir un mot de passe pour ce partage. Remplissez-la.
    * Le partage est alors activé


* Montage du partage sur l’ordinateur
    * Dans le Finder et dans le menu sélectionez *Aller* / *Se connecter au serveur…*
    * Utiliser *smb://minion0X.in-genomique.biologie.ens.fr* (remplacez minion0X par minion01 ou minion02 selon votre séquenceur. Appuyer sur le bouton *Se connecter*
    * Une boite de dialogue apparait. Choissez *Utilisateur référencé*, le nom de l’utilisateur est **minit** et le mot de passe celui que vous avez précédemment choisi. Appuyez sur le bouton *Se connecter*

**Exercice 3 : Utiliser ce partage pour copier des fichiers vers le montage. Vérifiez que les fichiers copiés sont bien visibles dans l’interface de MinKNOW**

### SFTP

SFTP est l’évolution sécurisée du protocole de transfert de fichier FTP qui date des années 70. SFTP est habituellement utilisable sur une machine dès qu’un serveur SSH est configuré, comme c’est le cas sur MinION Mk1C.

* Récupération d’un run. Ouvrir l’application `Terminal` de macOS et lancer les commandes suivantes (adapter la commande avec le nom de votre MinION : minion-0X -> minion-01 ou minion-02):
```bash
cd ~/Desktop
sftp -r minit@minion-0X.in-genomique.biologie.ens.fr:/data/TOTO .
```

* L’option `-r` de `sftp` active une copie récursive des dossiers/fichiers.

* Dépot de données sur le MinION (ex: pour relancer l’appel de base d'un run) :
```bash
cd ~/Desktop
sftp -rp TOTO minit@minion-0X.in-genomique.biologie.ens.fr:/data/TOTO-copie .
```

* L’option `-p` de `sftp` permet de préserver les permissions et les dates des fichiers.


<a name="minknow"></a>
## TP 4 : Interface de MinKNOW et lancement d'un run

Dans ce TP nous allons prendre en main MinKNOW, l'interface graphique permettant le contrôle du minion et du Mk1C. Cette interface permet l'accès à un certain nombre de paramètres, le lancement des runs, du basecalling, de l'alignements des lectures obtenues...etc


* Utilisation du MinION Mk1C via son interface graphique

Après lancement de l'application vous avez accès aux séquenceurs qui lui sont accessibles. Aujourd'hui, vous en voyez deux, minion01 (le minion via le MinIT) et minion02 (le Mk1C).
Choisissez le séquenceur sur lequel vous travaillez. Placez une flowcell dans l'emplacement prévu. Vous devez maintenant voir la flowcell que vous avez mis en place sur l'interface.


Le menu accessible sur la gauche de l'application vous propose 5 options : Start, Sequencing overview, Experiments, System messages, Host settings.
Parcourez les Host settings.

**Exercice 1 : Dans quel sous menu des settings devez vous aller pour redémarrer ou éteindre le système d'exploitation du Mk1C ou du MinIT ?
Où verifier que MinKNOW est bien à jour ?**

Ce sous menu vous permet de 
- savoir l’espace qu’il vous reste sur vos disques
- naviguer dans vos résultats, run par run
- effectuer la mise à jour du système d’exploitation de votre appareil 
- mettre à jour MinKNOW


* Vérification initiale du séquenceur ou de la flowcell

A la réception du séquenceur vous devez verifier son état. Pour le faire, vous trouverez une flowcell factice en plastique blanc dans la boite de l'appareil. Il s’agit de la flowcell de configuration (CTC). Insérez là dans l'emplacement de la flowcell et cliquez sur start. En choisissant la section Hardware check, vois pouvez lancer la vérification de votre matériel. 

**Exercice 2 : Lancez le Hardware Check**

Avant chaque lancement de run, vous devez aussi vous assurer que la flowcell rempli les conditions d’utilisation: il est nécessaire de vérifier le nombre de pores disponibles sur la flowcell avant de charger les échantillons.
Le nombre de pores disponibles doit être supérieur à : 
- 50 dans le cas d’une flowcell Flongle
- 800 dans le cas d’une flowcell MinION
Les flowcells sont remplacées si ce n’est pas le cas !

**Exercice 3 : Revenez au menu précedent et lancez le Flowcell Check**


* Paramétrage et lancement d’un run 

Il faut définir 
- Votre expérience 
- Le kit utilisé
- Le type de basecalling (choix du modèle de réseau de neurone) s’il est fait à la volée
- Les format de sortie de vos données
- L’alignement à la volée ou non et par conséquent, les séquences références

Commençons !
Définissez votre expérience et passez à la selection du kit (N’oubliez pas de lui donnez un nom !).

Dans la section permettant le choix du kit de séquençage à utiliser, tous les kits sont disponibles. Il est possible de les filtrer selon ce que l’on séquence, selon les banques faites…
Choisissez ce qui vous intéresse. A la plateforme nous utilisons le kit SQK-PBK004, C’est un kit ADN avec PCR.
Il est important de ne pas se tromper. Chaque kit possède des spécificités d’amorces et cet aspect sera primordial pour la partie basecalling, demultiplexing…

Passez au choix des options de runs.

Selon le type de séquençage que vous souhaitez faire, votre run va durer plus ou moins longtemps. 
Pour un RNASeq, un run de 72h est adapté. Si vous souhaitez tester la presence ou non d’une bactérie, 20 minutes peuvent suffire (votre flowcell peut être utilisée plusieurs fois).
Le voltage initial de la flowcell peut être modifié mais il vaut mieux être expert pour cela. 
Contrôle actif des canaux est enclenché ce qui autorise MinKNOW a monitorer les canaux en permanence pour une meilleure performance de ceux-ci. 
Le temps entre chaque changement des canaux est aussi paramétrable. Vous pouvez également sauvegarder un nombre de pore pour les faire intervenir dans la durée du run.
Concrètement, nous ne changeons jamais ces paramètres.

Passez à la configuration du basecalling.

 
**Configuration de l’appel de base**
L’appel de base peut être réalisé à la volée ou après le run. Il peut être réalisé sur le Mk1C, le MinIT ou un ordinateur indépendant.
Nous allons voir comment le lancer à la volée. Les paramètres importants restent les mêmes quelque soient la machine choisie pour réaliser l’appel de base.


Trois modes de basecalling sont possibles:
- Fast : Pratique pour le diagnostique parce rapide
- High-accuracy : Plus long mais moins d’erreur
- Modified : Dictionnaires de bases possibles incluent certaines bases modifiées

Passons aux code-barres:
Dans le  cas d’utilisation de code-barres, vous pouvez jouer sur plusieurs paramètres: 
- Suppression des code-barres aux extrémités des données basecallées
- Recherche des code-barres à chaque extrémité de la lecture pour classifier la lecture.
Si un seul des code-barres est trouvé, la lecture est perdue
- Recherche de code-barre au milieu de la lecture: Elimination de la lecture si un code barre est trouvé 

Attention, le sequençage nanopore est encore imprecis. Les sequences si elles sont petites comme des code-barres et qu'elles contiennent des erreurs peuvent etre mal reconnues. Vous risquez de perdre beaucoup à être trop stringent.



<a name="fast5"></a>
## TP 5 : Les fichiers bruts Fast5

Les fichiers Fast5 sont les fichiers contenant les données brutes du séquençage. Ces fichiers sont en fait des fichiers au format [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) qui utilise une structure arborescente propre à Nanopore.

Dans ce TP, nous allons explorer le contenu de ces fichiers Fast5 afin de mieux comprendre le principe le l’appel de base que nous aborderons dans le prochain TP.

Pour visualiser le contenu de ces fichiers, il est nécessaire d’utiliser des outils spécifiques. Dans le cadre de TP, nous utiliserons l’outil HDFView qui permet de visualiser des fichiers au format HDF5.

* Installation de HDFView
    * Aller dans le dossier *Outils* des documents de la formation présent sur le bureau de l’ordinateur
    * Faire un double clic sur *HDFView-3.1.2.dmg*
    * Acceptez les conditions d’utilisation
    * Une fenêtre avec une icone *HDFView* apparait, cliquez sur cette icône pour lancer l’application

Nous allons maintenant ouvrir un des fichiers Fast5 pour en visualiser le contenu

* Allez dans le menu *File* et sélectionnez *Open*. Appuyez sur le bouton *Option* et sélectionnez *All Files*. Sélectionnez un des fichiers Fast5 présent dans le dossier *Données* des documents de la formation présent sur le bureau de l’ordinateur.

**Question 1 : Combien y a-t-il d’éléments dans le panneau de gauche et pourquoi ?**

* Sélectionnez un des éléments du fichier et allez dans le sous-dossier *Raw* et double-cliquez sur le « fichier » *Signal*
* Une fenêtre apparaît avec un tableau contenant une seule colonne. Il s’agit des valeurs brutes du séquençage

**Question 2 : Combien y a-t-il de valeurs et quel est l’ordre de grandeur de celle-ci ?**

* Sélectionnez la première et seule colonne et appuyez ensuite le bouton en haut à gauche pour visualiser le signal. Une fenêtre avec les options de visualisation apparaît, appuyez simplement sur *OK*

**Question 3 : Identifiez la zone du signal correspondant à la queue polyA de la lecture (toutes les lectures n’en possèdent pas)**

**Question 4 : Dans les sous dossiers d’une lecture, retrouvez le numéro de pore, le numéro de la flowcell et la date de début de run**


<a name="basecalling-minknow"></a>
## TP 6: Relancer un appel de base avec MinKNOW

**TODO**


<a name="basecalling-cmdline"></a>
## TP 7 : Appel de Base en ligne de commande

L’appel de base est réalisé à l’aide du logiciel Guppy développé par Oxford Nanopore Technologies. D’autres outils existent/existaient pour cette tache, mais il est recommandé d’utiliser celui d’ONT qui certainement aujourd’hui le plus performant.

Dans ce TP, nous verrons comment lancer l’appel de base en ligne de commande et nous explorerons les fichiers générés lors de ce traitement.

Pour fonctionner, il est nécessaire de fournir à Guppy un fichier de configuration décrivant le modèle à utiliser pour effectuer l’appel de base.
Celui-ci peut être automatiquement déterminé par Guppy si on lui fournit à l’aide des arguments `--flowcell` et `--kit`.
Cependant si cette solution est choisie le mode *haute précision (hac)* sera automatiquement sélectionné.
Dans le cadre des données utilisées pour ce TP, ce sera la configuration *dna_r9.4.1_450bps_hac*. Afin de réduire les temps de calcul, nous forcerons l’utilisation de la configuration *dna_r9.4.1_450bps_fast* qui permettra d’effectuer l’appel de base en mode rapide.

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
                               --input_path /data/TOTO/no_sample/20210315_1508_MC-110337_0_FAO31058_dad08772/fast5 --fast5_out \
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
                          --input_path /data/TOTO/no_sample/20210315_1508_MC-110337_0_FAO31058_dad08772/fast5 --fast5_out \
                          --compress_fastq \
                          --recursive \
                          --min_score 40.000000 \
                          --config dna_r9.4.1_450bps_fast.cfg \
                          --barcode_kits SQK-PBK004
```

* Remarquera que cette ligne de commande précédente est plus lente (14m30s) que la précédente (7m30) car les paramètres de parallélisation n’ont pas été optimisés.

* Pour plus d’informations, notamment sur comment configurer l’alignement et le rognage des adaptateurs, il convient de se reporter à la [documentation de Guppy](https://community.nanoporetech.com/protocols/Guppy-protocol/v/gpb_2003_v1_revu_14dec2018) et à l’aide en ligne de commande (`guppy_basecaller --help`).

**Question 1 : Quel est l’interet d’utiliser Guppy en mode serveur sur un MinION Mk1C ? et sur GridION ou PromethION ?**

**Question 2 : Quel est le désavantage d’utiliser Guppy en mode serveur ?**

* À la fin de l’appel de base on obtient l’arborescence suivante
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

* Sur l’ordinateur, allez dans le dossier *formation-minion/appel_de_base* sur le Bureau et ouvrez avec Firefox le fichier *sequencing_telemetry.js*

**Question 7 : Que contient ce fichier ? Avons-nous vu déjà une partie de ces informations quelque part ?**

* Toujours dans le même dossier, ouvrez le fichier *sequencing_summary.txt* à l’aide d’un tableur (Microsoft Excel ou Libreoffice Calc), il s’agit d’un fichier texte tabulé (TSV).

**Question 8 : Que contient ce fichier ? Avons-nous vu un fichier avec le même nom précédemment ? Quelles sont les différences entre ces fichiers ?**

**Question 9 : Trouvez les colonnes pour identifier les lectures passant les filtres qualité, la longueur des lectures, la qualité moyenne des lectures et le code barre identifié**


<a name="qc"></a>
## TP 8 Contrôle Qualité post run

Dans ce dernier TP, nous comparons les rapports de contrôle qualité produits par différent outil et nous en comparerons les avantages et les inconvénients.

**Note : ** Pour des raisons de simplicité et de rapidité, nous utiliserons des rapports générés avant le début du TP. PycoQC et ToulligQC sont des outils qui s’installent très facilement et qui s’executent en quelques secondes/minutes.

### Rapport de MinKNOW

* À la fin d’un run, MinKNOW va sauver sous la forme d’un fichier PDF, les informations et les graphiques qu’il affichait au cours du run.
* Sur l’ordinateur, allez dans le dossier *formation-minion/qc/MinKNOW* sur le Bureau et ouvrez le rapport PDF.

**Question 1 : Que manque-t-il dans le rapport produit par MinKNOW à la fin du run ?**

### PycoQC

* PycoQC est un outil permettant de produire un rapport de contrôle qualité après démultiplexage. Il se base sur le contenu du fichier *sequencing_summary.txt*

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

* Sur l’ordinateur, allez dans le dossier *formation-minion/qc/PycoQC* sur le Bureau et ouvrez le rapport HTML.

**Question 2 : Qu’apporte PycoQC par rapport produit par MinKNOW ?**

### ToulliqQC

* ToulliqQC est un autre outil permettant de produire un rapport de contrôle qualité après démultiplexage. Il est développé par la plateforme génomique de l’ENS. Il se base sur les contenus des fichiers *sequencing_summary.txt* et *sequencing_telemetry.js*.

* ToulliqQC est un outil développé en Python. Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :

```bash
pip install toulliqQC
```

* Avec les données que nous avons générées, la commande à lancer pour produire le rapport est la suivante :

```bash
toulliqQC --report-name  Formation_MinION \
          --telemetry-source sequencing_telemetry.js \
          --sequencing-summary-source sequencing_summary.txt \
          --barcodes BC11,BC12 \
          --output .
```

* Sur l’ordinateur, allez dans le dossier *formation-minion/qc/ToulligQC* sur le Bureau et ouvrez le rapport HTML.

**Note :** Le rapport produit ici a été réalisé par la version beta 3 de ToulligQC 2.0. La version finale est attendue d’ici la fin du mois.

**Question 3 : Qu’apporte ToulligQC par rapport produit par PycoQC ?**

**Question 4 : Que manque-t-il à ToulligQC ?**

### Nanocomp

**TODO**


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
