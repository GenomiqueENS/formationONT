<a href="https://genomique.biologie.ens.fr/"><img src="https://github.com/GenomiqueENS/eoulsan/raw/master/src/site/resources/images/logo-GenomiqueENS-90pxh.png" align="left"></a>
# Formation MinION Session Bioinformatique<br/>


Formateurs : Salomé Brunon (*brunon@bio.ens.psl.eu*) Ali Hamraoui (*hamraoui@bio.ens.psl.eu*), Laurent Jourdren (*jourdren@bio.ens.psl.eu*) et Sophie Lemoine (*slemoine@bio.ens.psl.eu*)

Contact Plateforme GenomiqueENS :

* Site Web [https://genomique.biologie.ens.fr](https://genomique.biologie.ens.fr/)
* Courriel [genomique@bio.ens.psl.eu](mailto:genomique@bio.ens.psl.eu)
* Twitter/X [@Genomique_ENS](https://twitter.com/Genomique_ENS)

## Sommaire

* Présentation de la Plateforme Génomique de l'ENS
* Rappel du principe de la technologie ONT
* Présentation des différents types de flowcells et séquenceur ONT
* Principe de l’appel de base
* [Gestion du P2 solo](#p2solo)

* [TP 1 : Interface de MinKNOW, lancement d'un run et du basecalling via l'interface](#minknow)
* [TP 2 : Utilisation d'une sample sheet](#samplesheet)
* [TP 3 : Utilisation de paramètres enregistrés](#settings)
* [TP 4 : Les fichiers bruts Fast5](#fast5)
* [TP 5 : Les fichiers bruts POD5](#pod5)
* [TP 6 : Appel de Base en ligne de commande](#basecalling-cmdline)
* [TP 7 : Contrôle Qualité post run](#qc)
* Alignement
* [TP 8 : Création d'un index pour minimap2 avec l'interface graphique](#index-gui)
* [TP 9 : Faire un alignement avec l'interface graphique](#align-gui)
* [TP 10 : Faire un alignement en ligne de commande](align-cli)
* Conclusion
* [Bibliographie](#biblio)


#### Informations utiles

Lors de ce TP, nous utilisons deux séquenceur MinION et un séquenceur PromethION P2 solo:

* Un MinION Mk1B couplé à un PC sous Ubuntu 20.04 (6 coeurs, 32 Go RAM, NVIDIA GeForce RTX 2080Ti)
* Un MinION Mk1B et un PromethION P2 solo couplé à un PC sous Ubuntu 20.04 (32 coeurs, 256 Go RAM, 2 x NVIDIA RTX A6000)

MinKNOW est le logiciel pilotant les séquenceurs MinION, GridION et PromethION.
Au cours de ce TP, la version de MinKNOW utilisé est la 24.02.10 datant du 27 mars 2024.
Les ordinateurs pour cette formation fonctionne sous Ubuntu 20.04 mais le comportement de MinKNOW est similiare sous macOS et Windows 10 ou 11.

Les données utilisées lors de ce TP sont celles qui ont été produites lors d'un run le 19 septembre 2023 sur la plateforme GenomiqueENS.
Le run effectué lors de la session expérimentale de la formation se nommait *20230919_P2solo-cDNA-R9_C2023*, un peu plus 14 millions lectures (ADN) avaient été produites en utilisant une flowcell de type *FLO-MIN106* (R9.4.1) et le kit *SQK-PCB111-24*.
Six échantillons avait été multiplexé, les codes barres utilisés sont les suivants : BP08, BP09, BP10, BP11, BP12 et BP13.
Afin de réduire les temps de calcul, seul 40 000 lectures de ce run seront utilisées lors des différents TP.


**Note :** Le MinION Mk1C n'étant désormais plus commercialisé par Nanopore, nous n'aborderons pas lors de cette formation l'administration système et la gestion des données pour les séquenceurs MinION Mk1C, GridION et PromethION P2i.
Si ces thématiques vous intéressent, reportez-vous aux [supports de cours de cette formation réalisée en 2023](https://github.com/GenomiqueENS/formationONT/tree/2023).

<a name="p2solo"></a>
## Gestion du PromethION P2 solo

Le séquenceur P2 solo peut être assimilé à un MinION Mk1B, le logiciel MinKNOW installé sur un PC fonctionne de la même manière que pour un MinION Mk1B.
C'est pour cela que dans le reste de ce document, le PromethION P2 solo ne sera que peu évoqué.

Le domaine où le PromethION P2 solo diffère grandement du MinION Mk1B concerne les prérequis matériels pour le faire fonctionner :
- Ubuntu 20.04, macOS 10.14 (Mojave) ou Windows 10
- 2 To SSD interne + 6-8 To SDD pour les données
- NVIDIA RTX 3080 avec 12 Go / NVIDIA RTX A6000
- 64 Go RAM
- 8-12 cores (Intel i7 7ème génération ou AMD Ryzen)
- USB 5 Gb/s

Chaque run va nécessiter 2 à 3 To pour les données (ultra) brutes (Fast5/POD5) ainsi que des GPU puissants pour réaliser l'appel de base.
Il faut donc bien prendre en considération les besoins matériels et en administration système (gestion des données une fois produites) pour prendre en charge correctement un PromethION P2 solo.



<a name="minknow"></a>
## TP 1 : Interface de MinKNOW, lancement d'un run et de l'appel de base via l'interface

Dans ce TP nous allons prendre en main MinKNOW, l'interface graphique permettant le contrôle du MinION Mk1b et du PromethION P2 Solo.
Cette interface permet l'accès à un certain nombre de paramètres tels que les lancement des runs, du basecalling, de l'alignements des lectures obtenues, etc…


### Utilisation des MinION et du PromethION via leurs interfaces graphiques

Après lancement de l'application vous avez accès aux séquenceurs qui lui sont accessibles.
Choisissez le séquenceur sur lequel vous travaillez.
Placez une flowcell dans l'emplacement prévu.
Vous devez maintenant voir la flowcell que vous avez mis en place sur l'interface.

Le menu accessible sur la gauche de l'application vous propose 4 options :
- Start
- Sequencing overview
- Experiments
- System messages


### Vérification initiale du séquenceur ou de la flowcell

À la réception du séquenceur vous devez vérifier son état.
Pour le faire, vous trouverez une flowcell factice en plastique blanc dans la boite de l'appareil.
Il s’agit de la flowcell de configuration (CTC).
Insérez là dans l'emplacement de la flowcell et cliquez sur start.
En choisissant la section Hardware check dans la section *Start*, vous pouvez lancer la vérification de votre matériel.

**Exercice 1 : Lancez le Hardware Check**

Avant chaque lancement de run, vous devez aussi vous assurer que la flowcell rempli les conditions d’utilisation: il est nécessaire de vérifier le nombre de pores disponibles sur la flowcell avant de charger les échantillons.
Le nombre de pores disponibles doit être supérieur à :

- 50 dans le cas d’une flowcell Flongle
- 800 dans le cas d’une flowcell MinION
- 5000 dans le cas d’une flowcell PromethION

Les flowcells sont remplacées si ce n’est pas le cas !

**Exercice 2 : Revenez au menu précédent et lancez le Flowcell Check**

### Lancement d'un run

**Exercice 3 : Paramétrez et lancez votre run**

Il faut définir :

- Votre expérience
- Le kit utilisé
- Le type de basecalling (choix du modèle de réseau de neurone) s’il est fait à la volée
- Les format de sortie de vos données
- L’alignement à la volée ou non et par conséquent, les séquences références

Commençons !


**Définissez votre expérience et passez à la sélection du kit (N’oubliez pas de lui donner un nom !) :**

Dans la section permettant le choix du kit de séquençage à utiliser, tous les kits sont disponibles.
Il est possible de les filtrer selon ce que l’on séquence, selon les banques faites…
Choisissez ce qui vous intéresse.
Sur notre plateforme nous utilisons le kit SQK-PBK004, C’est un kit ADN avec PCR.
Il est important de ne pas se tromper : chaque kit possède des spécificités d’amorces et cet aspect sera primordial pour l'appel de base, le démultiplexage…
Attention, la chimie est en ce moment en cours d'évolution et les kits ne sont pas tous disponibles selon les Flowcells et les séquenceurs.


**Passez au choix des options de runs :**

Selon le type de séquençage que vous souhaitez faire, votre run va durer plus ou moins longtemps.
Pour un RNASeq, un run de 72 h est adapté.
Si vous souhaitez tester la présence ou non d’une bactérie, 20 minutes peuvent suffire (votre flowcell peut être utilisée plusieurs fois).
Le voltage initial de la flowcell peut être modifié mais il vaut mieux être expert pour cela.
Le contrôle actif des canaux est enclenché ce qui autorise MinKNOW à monitorer les canaux en permanence pour une meilleure performance de ceux-ci.
Le temps entre chaque changement des canaux (mux scan) est aussi paramétrable.i
Vous pouvez également sauvegarder un pourcentage de pores pour les faire intervenir dans la durée du run.
C'est dans cette section que vous pouvez parametrer MinKNOW pour qu'il fasse de l'adaptive sampling en enrichissant ou en rejetant certaines séquences. 
Dans ce cas vous devez fournir une séquence FASTA de référence (type génome) et un fichier BED de ce que vous voulez enrichir ou rejeter.
Vous pouvez également spécifier des code-barres à enrichir.
Il faut noter que ces deux possibilités, adaptive sampling ou barcode balancing, sont en version beta.
Vous pouvez jouer avec ces options pendant le TP.


**Passez à la configuration du basecalling :**

L’appel de base peut être réalisé à la volée ou après le run.
Il peut être réalisé sur la machine d'acquisition ou un ordinateur indépendant.
Nous allons voir comment le lancer à la volée.
Les paramètres importants restent les mêmes quelque soit la machine choisie pour réaliser l’appel de base.

Deux modes de basecalling sont possibles :

- Fast basecalling (fast) : Pratique pour le diagnostique parce rapide
- High-accuracy basecalling (hac) : Plus long mais moins d’erreur
- Super-accurate basecalling : Prend tout son temps pour obtenir l'appel de base le plus précis possible

**Note :** En mode *super acurrate basecalling (sup)* il est indispensable de disposer d'une carte GPU puissante pour réaliser l'appel de base.

Il est nécessaire de connaitre deux élements du run :
- Le type de flow cell (ex: FLO-MIN106)
- Le type de chimie sur la flow cell (ADN ou ARN)


**Passons aux codes-barres :**

Dans le  cas d’utilisation de codes-barres, vous pouvez jouer sur plusieurs paramètres :

- Suppression des codes-barres aux extrémités des données basecallées (*Trim barcodes*)
- Recherche des codes-barres à chaque extrémité de la lecture pour classifier la lecture : si un seul des code-barres est trouvé, la lecture est perdue (*Barcode both ends*)
- Recherche de codes-barres au milieu de la lecture : Élimination de la lecture si un code barre est trouvé (*Mid-read barcode filtering*)
- Filtrage des codes-barres selon leur score de façon à etre plus stingent sur leur qualité

**Attention :** le séquençage nanopore est encore imprecis.
Les séquences si elles sont petites comme des code-barres et qu'elles contiennent des erreurs peuvent être mal reconnues.
Vous risquez de perdre beaucoup à être trop stringent.


**Lancement de l'alignement à la volée :**

MinKNOW  peut lancer l’alignement à la volée.
Minimap2 est le mapper qui est utilisé de façon standard.
Si vous souhaitez le faire, vous devez fournir un fichier FASTA de référence.
Si vous faites du RNA-seq, vous pouvez également donner en entrée de minimap2, un fichier BED12 définissant les jonctions de vos isoformes.
Vous pouvez utiliser paftools, un outil intégrer à minimap2, pour construire les BED12 correspondant à votre problématique à partir des fichiers d’annotation GTF qui sont plus courant.


**Quels sont les fichiers de sorties à choisir en sortie de MinKNOW ?**

- Des Fast5 : Ce sont les données brutes (par défaut pour les runs R9.4.1).
Il est important de les conserver si l’on veut relancer le basecalling en fonction des évolutions de Dorado
- Des POD5 : Ce sont les données brutes (par défaut pour les runs R10.4.1).
Il s'agit d'un nouveau format permettant une meilleure compression des données brutes.
- Des FASTQ : Ce sont les données basecallées, demultiplexées (si besoin) et classées en pass/fail
- Des BAM : Ce sont les données alignées si l’alignement à la volée a été demandé

Vous pouvez choisir le critère qui classera la lecture en pass ou fail.
Classiquement, les lectures aillant un score de qualité inférieur à 8 en mode "fast" sont considérées comme mauvaises (fail).
La valeur par défaut de ce seuil change selon le type d'appel de base (fast : 8, hac : 9 et sup : 10).
Les lectures peuvent être filtrées sur leur qscore minimal et/ou leur taille.
Vous pouvez aussi choisir de couper les lectures chimériques (voir section suivante) formées au moment du séquençage *Enable read splitting*.


**Quid du fichier Fast5 Bulk ?**

Dans ce type de fichier Fast5, MinKNOW ne fait pas de coupure entre chaque lecture d’un pore (*Advanced options*):

- elles restent liées en une longue séquence comprenant les adaptateurs et les séquences d'interet.
- il est possible de visualiser le signal et de voir les coupures déterminant les lectures dans BulkVis par exemple [Publi de Bulkvis].
Selon la séquence, il est possible que MinKnow ne coupe pas au bon endroit.
Des chimères peuvent être créées de cette façon.

Attention, cette option génère un gros volume de données.


<a name="samplesheet"></a>
## TP 2 : Utilisation d'une samplesheet


L'utilisation d'une sample sheet permet aux utilisateurs d'automatiser la configuration du lancement d'un ou de plusieurs run simultanément.
Il s'agit de fichiers texte au format CVS (Comma-separated values) contenant sous un forme d'un tableau différents information concernant le run :
- Le nom de l'expérience (champ "experiment_id")
- L'échantillon (champ "sample_id")
- Le type de flow cell (champ "flow_cell_product_code")
- Le numéro de série de la flow cell (champ "flow_cell_id")
- La position de la flow cell sur le séquenceur (champ "position_id")
- Le kit utilisé (champ "kit")
- Le ou les échantillons séquencés ainsi que leurs code barres (champs "barcode" et "kit")



* Exemple de sample sheet minimale pour un séquenceur avec une seule position (MinION)

| flow_cell_id | experiment_id | flow_cell_product_code | kit           |
| ------------ | ------------- | ---------------------- |-------------- |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 |



* Exemple de sample sheet pour un séquenceur avec plusieurs position (GridION, PromethION)

| flow_cell_id | position_id | experiment_id | sample_id      | flow_cell_product_code | kit           |
| ------------ | ----------- | ------------- | -------------- | ---------------------- |-------------- |
| CTC84999     | P2S-01171-A | Bidon_A2024   | MonEchantillon | FLO-PRO002             | SQK-PCB111-24 |



* Exemple sample sheet pour un séquenceur avec une seule position avec des codes barres

| flow_cell_id | experiment_id | flow_cell_product_code | kit           | barcode   | alias    |
| ------------ | ------------- | ---------------------- |-------------- | --------- | -------- |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 | barcode01 | WT01     |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 | barcode02 | WT02     |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 | barcode03 | WT03     |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 | barcode04 | Mutant01 |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 | barcode05 | Mutant02 |
| FAK02081     | Bidon_A2024   | FLO-MIN106             | SQK-PCB111-24 | barcode06 | Mutant03 |


Les champs "experiment_id", "flow_cell_product_code", "kit" et "flow_cell_id" ou "position_id" sont obligatoires.
Des champs non définis dans la spécification de Nanopore peuvent être utilisés mais seront sans effet sur MinKNOW ou tout autre logiciel (ils ne provoqueront pas d'erreur).


La spécification du format des fichiers sample sheet décrit des champs non évoqué ici ("type", "internal_barcode", "external_barcode"...), vous pouvez retrouver
les spécifications complete du format en ligne dans la documentation de [MinKNOW](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/experiment-companion-minknow/v/mke_1013_v1_revdc_11apr2016/sample-sheet-upload).

Le plus simple pour créer de tel fichiers est d'utiliser un tableur.
Cependant, il faut bien faire attention au moment de sauver le fichier au format CSV car dans les pays où le séparateur décimal est la virgule, le caractère ";" sera utilisé comme séparateur entre les champs au lieu de la virgule.


Pour importer un fichier sample sheet dans MinKNOW, appuyer sur Start puis sur le boutton "⁝ More" et selectionner "Sample sheet import".
Choisir ensuite un fichier sample sheet CSV dans le champ "Select a CSV file to import".
MinKNOW va aller vérifier si la sample sheet est correcte.
Si c'est le cas, le ou les runs vont pouvoir être lancés.

**Attention** : En mode sample sheet MinKNOW ne proposera d'interface graphique pour configurer le reste de la configuration d'un run (paramètres de sauvegarde des données brutes, basecalling, demultiplexage et alignement).
MinKNOW utilisera les paramètres par defaut.
Cependant, il est possible de définir ces autres paramètres à l'aide d'un fichier "settings".



Question 1 : À partir du premier exemple de sample sheet, ouvrir un tableur (Libreoffice Calc sous Linux) faire un fichier CSV et importer le dans MinKNOW.
Question 2 : Faire un fichier CSV valide utilisant toutes les positions disponibles des séquenceurs connecter à votre ordinateur. Le fichier samplesheet doit posseder un champ "sample_id" et un nombre de codes barres différent sur chaque flow cell.


<a name="settings"></a>
## TP 3 : Utilisation de paramètres enregistrés


Comme nous l'avons vu précédement, il peut être utile de réutiliser les paramètres de sauvegarde des données brutes, basecalling, demultiplexage et alignement utilisés pour plusieurs runs.
Ces paramètres (settings) sont écrits dans un fichier au format JSON une fois sauvegardés.
Le seul moyen à l'heure actuelle de créer ces fichiers à l'heure actuelle est un éditeur de texte.
La [documentation de MinKNOW](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/experiment-companion-minknow/v/mke_1013_v1_revdc_11apr2016/sample-sheet-upload) permet de connaitre les différents paramètres et leur valeur par défaut.
Les valeurs par défaut des paramètres seront utilisées si les paramètres sont abscents du fichier "settings".

* Exemple de fichier "settings" :
```json
{
    "runLengthHours": 72,
    "basecallModel": "dna_r9.4.1_450bps_fast.cfg",
    "barcodingEnabled": true,
    "trimBarcodesEnabled": true,
    "fastQEnabled": true,
    "pod5Enabled": false,
}
```

Les paramètres enregistrés peuvent désormais être utiliser pour lancer un ou plusieurs runs à l'aide d'une sample sheet.


<a name="fast5"></a>
## TP 4 : Les fichiers bruts Fast5

Les fichiers Fast5 sont les fichiers contenant les données brutes du séquençage.
Ces fichiers sont en fait des fichiers au format [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) qui utilise une structure arborescente propre à Nanopore.

Pour visualiser le contenu de ces fichiers, il est nécessaire d’utiliser des outils spécifiques.
Dans le cadre de TP, nous utiliserons l’outil HDFView qui permet de visualiser des fichiers au format HDF5.

**Note :** Depuis la version 21.05.8 de MinKNOW (juin 2021), le format des fichiers Fast5 a évolué.
Le signal électrique est désormais compressé par défaut à l'aide de l'algorithme [VBZ](https://github.com/nanoporetech/vbz_compression) développé par ONT.
Cet algorithme permet de réduire d'environ 1/3 la taille des données précédement compressées à l'aide de l'algorithme gzip et de réduire très fortement les temps de compression/décompression.
Cet algorithme n'étant pas standard, il faut installer un greffon (plugin) afin de pouvoir accèder aux données du signal electrique avec les outils standard HDF5.
Ce greffon est disponible sur la page GitHub du projet [vbz_compression](https://github.com/nanoporetech/vbz_compression).
Pour ce TP, le greffon a été préinstallé sur les postes de travail.

Dans ce TP, nous allons explorer le contenu de ces fichiers Fast5 afin de mieux comprendre le principe le l’appel de base que nous aborderons dans un prochain TP.

* Installation de HDFView sous Linux
    * Aller dans le dossier *Outils* des documents de la formation présent sur le bureau de l’ordinateur
    * Ouvrir le dossier *HDFView-Linux*
    * Faire un double clic sur *hdfview.sh* puis cliquer sur *Lancer dans un terminal*

Nous allons maintenant ouvrir un des fichiers Fast5 pour en visualiser le contenu

* Allez dans le menu *File* et sélectionnez *Open*
* Appuyez sur le bouton *Option* et sélectionnez *All Files*
* Sélectionnez un des fichiers Fast5 présent dans le dossier *Données* des documents de la formation présent sur le bureau de l’ordinateur.

**Question 1 : Combien y a-t-il d’éléments dans le panneau de gauche et pourquoi ?**

* Sélectionnez un des éléments du fichier et allez dans le sous-dossier *Raw* et double-cliquez sur le « fichier » *Signal*
* Une fenêtre apparaît avec un tableau contenant une seule colonne.
Il s’agit des valeurs brutes du séquençage
* Sélectionnez la première et seule colonne et appuyez ensuite le bouton en haut à gauche pour visualiser le signal.
Une fenêtre avec les options de visualisation apparaît, appuyez simplement sur *OK*

**Question 2 : Identifiez la zone du signal correspondant à la queue polyA de la lecture (toutes les lectures n’en possèdent pas)**

**Exercice 3 : Dans les sous dossiers d’une lecture, retrouvez le numéro de pore, le numéro de la flowcell et la date de début de run**


<a name="pod5"></a>
## TP 5 : Les fichiers bruts POD5

Le format Fast5 n'est plus le format de fichier de sortie par défaut depuis la mi-mai 2023 pour les runs utilisant le Kit 14.
Le format POD5 est le remplaçant du format Fast5.
Il est bien plus performant dans la manière de stocker les données et permet notamment de réaliser bien plus efficacement l'appel de base, notamment en duplex.


* Installation de la commande pod5:
```bash
pip3 install pod5
```

* Afficher les metadonnées d'un run
```bash

# Se placer dans le dossier où sont les données
cd ~/Bureau/formation/Données/pod5

inspect debug  FAW68019_c16beb57_cd2d48c7_0.pod5
``

**Question 1 : Retrouver la date du run, le type de séquenceur, le type de flowcell et le type de kit ?**

* Afficher un résumé d'un fichier POD5
```bash

# Se placer dans le dossier où sont les données
cd ~/Bureau/formation/Données/pod5

# Créer un fichier résumé
pod5 view  FAW68019_c16beb57_cd2d48c7_0.pod5 --output ~/Bureau/pod5-summary.tsv

# Visualiser ce fichier résumé (appuyer sur la touche "q" pour quitter, la touche "_" permet d'agrandir la taille d'une colonne). Vous pouvez également utiliser un tableur pour cela
vd ~/Bureau/pod5-summary.tsv
```

**Question 2 : Combien y-a-t-il de lignes dans ce fichier ?**
**Question 3 : Retrouver les informations correspondant à la date de début du signal et du "channel" d'une lecture ?**


* Conversion de fichiers POD5 en Fast5
```bash

# Se placer dans le dossier où sont les données
cd ~/Bureau/formation/Données

# Convertir les fichiers POD5 du dossier "pod5" en fichiers Fast5 dans le dossier "fast5"
pod5 convert to_fast5  pod5/*.pod5 --output fast5/

# Afficher la liste des fichiers nouvellement crées
ls -ltr fast5/
```

* Conversion de fichiers POD5 en Fast5
```bash

# Se placer dans le dossier où sont les données
cd ~/Bureau/formation/Données

# Créer le dossier de destination
mkdir pod5-from-fast5

# Convertir les fichiers Fast5 du dossier "fast5" en fichiers POD5 dans le nouveau dossier "pod5-from-fast5"
pod5 convert fast5  fast5/*.fast5 --output pod5-from-fast5/

# Afficher la liste des fichiers nouvellement crées
ls -ltr pod5-from-fast5/
```


<a name="basecalling-cmdline"></a>
## TP 6 : Appel de Base en ligne de commande

L’appel de base est l'étape au cours de laquelle où le signal électrique est traduit en séquences nucléotidiques.
Au cours de cette étape, on passe de données dans un format Fast5/POD5 à des données au format FASTQ.
L’appel de base est réalisé à l’aide du logiciel Guppy développé par Oxford Nanopore Technologies.
D’autres outils existent/existaient/existeront (Guppy va être remplacé par Dorado d’ici quelques mois) pour cette tâche, mais il est recommandé d’utiliser celui d’ONT qui certainement aujourd’hui le plus performant.
Oxford Nanopore propose également d’autres logiciels pour l’appel de base mais ceux-ci sont dédiés à la recherche algorithmique et il ne vaut mieux pas les utiliser en production.

Dans ce TP, nous verrons comment lancer l’appel de base en ligne de commande et nous explorerons les fichiers générés lors de ce traitement.

Pour fonctionner, il est nécessaire de fournir à Guppy un fichier de configuration décrivant le modèle à utiliser pour effectuer l’appel de base.
Celui-ci peut être automatiquement déterminé par Guppy si on lui fournit à l’aide des arguments `--flowcell` et `--kit`.
Cependant si cette solution est choisie le mode *haute précision (hac)* sera automatiquement sélectionné.
Dans le cadre des données utilisées pour ce TP, ce sera la configuration *dna_r9.4.1_450bps_hac*.
Afin de réduire les temps de calcul, nous forcerons l’utilisation de la configuration *dna_r9.4.1_450bps_fast* qui permettra d’effectuer l’appel de base en mode rapide.

**Note 1 :** Avec Dorado, le nouveau "basecaller" de Nanopore, les données en sortie de l'appel de base seront stockées au format BAM.
Le format BAM est à l'origine un format de fichier dédié au stockage d'alignements de séquences sur un génome de référence.
Il peut toutefois être utilisé pour stocker des séquences qui n'ont pas été alignées.

**Note 2 :** Si vous disposez d'une carte GPU puissante (non recommandé sur un MinION Mk1C), vous pouvez également forcer l'utilisation du mode super accuracy à l'aide de la configuration *dna_r9.4.1_450bps_sup*.
La qualité de l'appel de base en mode super accuracy est légèrement supérieure au mode high accuracy mais demande beaucoup plus de temps de calcul que ce dernier.
Le mode high accuracy est généralement un bon compromis entre temps de calcul et qualité de l'appel de base.

* Ouvrez le logiciel `Terminal` sur l'ordinateur et connectez-vous via `ssh` au séquenceur

* Pour connaître les fichiers de configuration qui seront automatiquement sélectionnés en fonction de la flowcell et du kit, il suffit de lancer la commande suivante :
```bash
guppy_basecaller --print_workflows
```

* L’appel de base peut-être lancé en ligne de commande de la manière suivante sur le MinION Mk1C (le dossier de sortie doit exister):
```bash
mkdir /data/appel_de_base_ligne_de_commande_guppy_server
/opt/ont/guppy/bin/guppy_basecall_client \
                                         --port ipc:///tmp/.guppy/5555 \
                                         --server_file_load_timeout 600 \
                                         --save_path /data/appel_de_base_ligne_de_commande_guppy_server \
                                         --config dna_r9.4.1_e8.1_fast_mk1c.cfg \
                                         --progress_stats_frequency 2 \
                                         --input_path /data/TOTO/no_sample/20210315_1508_MC-110337_0_FAO31058_dad08772/fast5/ \
                                         --compress_fastq \
                                         --recursive \
                                         --barcode_kits SQK-PBK004
```

**Note :** Pour réaliser l'appel de base en mode serveur en utilisant le GPU avec le PromethION P2 solo, il est nécessaire de réaliser une [configuration additionnelle de MinKNOW](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/promethion-2-solo-user-manual/v/p2s_9172_v1_revh_14oct2022/installing-gpu-version-of-guppy-with-minknow-for-minion).

* À titre d’information, on peut également lancer Guppy hors mode serveur avec la ligne de commande suivante (il faut retirer l’option `--device` si vous ne disposez pas d’un GPU) :

```bash
mkdir /data/appel_de_base_ligne_de_commande_guppy
/opt/ont/guppy/bin/guppy_basecaller \
                                    --device auto \
                                    --save_path /data/appel_de_base_ligne_de_commande_guppy \
                                    --config dna_r9.4.1_e8.1_fast_mk1c.cfg \
                                    --input_path /data/TOTO/no_sample/20210315_1508_MC-110337_0_FAO31058_dad08772/fast5/ \
                                    --compress_fastq \
                                    --recursive \
                                    --barcode_kits SQK-PBK004
```

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

**Question 4 : Deux échantillons ont été déposés sur la flowcell lors du run.
Pourquoi avons-nous une 12 dossiers pour des codes-barres lieu de 2 ?**

**Question 5 : À quoi correspond le dossier les fichiers FASTQ du dossier *unclassified* ?**

**Exercice 6 : Ouvrez un des fichiers Fast5 produits lors du démultiplexage avec HDFView.
Comparez leur structure avec celle des fichiers Fast5 avant démultiplexage.
Retrouvez les séquences appelées au format FASTQ dans les fichiers Fast5.**

* Sur l’ordinateur, allez dans le dossier *Formation-MinION/appel_de_base* sur le Bureau et ouvrez avec Firefox le fichier *sequencing_telemetry.js*

**Question 7 : Que contient ce fichier ? Avons-nous vu déjà une partie de ces informations quelque part ?**

* Toujours dans le même dossier, ouvrez le fichier *sequencing_summary.txt* à l’aide d’un tableur (Microsoft Excel ou Libreoffice Calc), il s’agit d’un fichier texte tabulé (TSV).

**Question 8 : Que contient ce fichier ? Avons-nous vu un fichier avec un nom identique précédemment ? Quelles sont les différences entre ces fichiers ?**

**Question 9 : Trouvez les colonnes pour identifier les lectures passant les filtres qualité, la longueur des lectures, la qualité moyenne des lectures et le code barre identifié**


**Note 3 :** Avec Dorado, le nouveau "basecaller" de Nanopore, le fichier *sequencing_summary.txt* n'est plus automatiquement généré lors de l'appel de base.
Pour créer ce fichier il faudra utiliser la commande `dorado summary`.

<a name="qc"></a>
## TP 7 Contrôle Qualité post run

Dans ce dernier TP, nous comparons les rapports de contrôle qualité produits par différents outils et nous en comparerons les avantages et les inconvénients.

**Note :** Pour des raisons de simplicité et de rapidité, nous utiliserons des rapports générés avant le début du TP.
PycoQC et ToulligQC sont des outils qui s’installent très facilement et qui s’exécutent en quelques secondes/minutes.

### Rapport de MinKNOW

* À la fin d’un run, MinKNOW va sauver sous la forme d’un fichier PDF, les informations et les graphiques qu’il affichait au cours du run.
* Sur l’ordinateur, allez dans le dossier *Formation-MinION/qc/MinKNOW* sur le Bureau et ouvrez le rapport PDF.

**Question 1 : Que manque-t-il dans le rapport produit par MinKNOW à la fin du run ?**

### PycoQC

* PycoQC est un outil permettant de produire un rapport de contrôle qualité après démultiplexage.
Il se base sur le contenu du fichier *sequencing_summary.txt*.

* PycoQC est un outil développé en Python.
Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :
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

* ToulliqQC est un autre outil permettant de produire un rapport de contrôle qualité après démultiplexage.
Il est développé par la plateforme génomique de l’ENS.
Il se base sur les contenus des fichiers *sequencing_summary.txt* et *sequencing_telemetry.js*.

* ToulliqQC est un outil développé en Python.
Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :

```bash
pip install toulligqc
```
ToulligQC peut être également installé via Docker ou Conda.

* Avec les données que nous avons générées, la commande à lancer pour produire le rapport est la suivante :

```bash
toulligqc --report-name Formation_MinION \
          --telemetry-source sequencing_telemetry.js \
          --sequencing-summary-source sequencing_summary.txt \
          --barcodes BC11,BC12 \
          --output .
```

* Sur l’ordinateur, allez dans le dossier *Formation-MinION/qc/ToulligQC* sur le Bureau et ouvrez le rapport HTML.

**Note :** Le rapport produit ici a été réalisé par la version 2.3 de ToulligQC.

**Question 3 : Qu’apporte ToulligQC par rapport produit par PycoQC ?**

**Question 4 : Que manque-t-il à ToulligQC ?**

### Nanocomp

NanoComp est un outil faisant partie d'une suite appelée NanoPack.
Il permet de comparer des echantillons d'un même run via le fichier sequencing_summary.txt, des fichiers FASTQ, des FASTA, des alignements via des fichiers BAM.
Il est donc utilisable à plein d'étapes de l'analyse se qui en fait un outil intéressant.
La suite est toujours maintenue.
NanoPack est un outil développé en Python.
Pour l’installer, il suffit de lancer la commande suivante (la commande `pip` est remplacé dans certaines distributions Linux par `pip3` pour la version Python 3 de pip qui doit être utilisée pour installer l’outil) :

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


<a name="index-gui"></a>
## TP 8 : Création d'un index pour minimap2 avec l'interface graphique

- Appuyer sur Start > ⁝ More > Create .mmi from .fasta
- Select an input .fasta file : chemin vers le fichier .fasta
- Select a folder for the output .mmi file : dossier de sortie ou sera créer l'index
- Appuyer sur start
- Message : ".fasta file has been processed"



<a name="align-gui"></a>
## TP 10 : Faire un alignement avec l'interface graphique


<a name="align-cli"></a>
## TP 11 : Faire un alignement en ligne de commande


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

