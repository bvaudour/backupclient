# backupclient

Script de sauvegarde de projets hébergés sur des serveurs SSH

## Usage

```
backupclient [options]

Options:
  -d  sauvegarde journalière (par défaut si aucun argument n'est défini)
  -w  sauvegarde hebdomadaire
  -m  sauvegarde mensuelle
```

## Fonctionnement

backupclient est conçu pour pouvoir être lancé via des cronjobs. Il peut, pour chaque projet défini dans la configuration, sauvegarder les fichiers et les BDD associées en passant par un serveur SSH.

Concernant les BDD, un dump complet est effectué quelque soit le type de sauvegarde (journalière, hebdomadaire ou mensuelle). À la fin d’une sauvegarde réussie, les anciennes sauvegardes sont supprimées suivant la configuration des variables de rotation.

Concernant les fichiers, une sauvegarde complète est systématiquement effectuée pour les sauvegardes mensuelles. Sinon, il s’agit d’une sauvegarde différentielle (identifiée par un nom d’archive terminant par .diff.tar.gz) par rapport à la dernière sauvegarde complète. À la fin d’une sauvegarde réussie, les anciennes sauvegardes sont supprimées suivant la configuration de rotation (en cas de sauvegarde différentielle, la sauvegarde complète précédente est conservée tant qu’il reste des sauvegardes différentielles associées).

## Fichiers/dossiers utilisés

* **backupclient** : exécutable
* **/var/log/backupclient/** : dossier contenant les logs de sauvegarde
* **/etc/backupclient/** : dossier contenant les fichiers de configuration. Il contient les éléments suivants :

    - **backup.conf** : fichier de configuration général
    - **model.conf** : modèle de fichier de configuration spécifique à un projet
    - **available/** : répertoire contenant les fichiers de conf spécifiques
    - **enabled/** : répertoire contenant les liens vers les fichiers de conf spécifiques activés

## Variables de configuration

### Variables générales

* **C_backupdir** : Répertoire de destination des sauvegardes
* **D_destdir** : Sous-répertoire pour la sauvegarde des fichiers
* **M_destdir** : Sous-répertoire pour la sauvegarde des BDD
* **C_daily** : Sous-répertoire pour la sauvegarde quotidienne
* **C_weekly** : Sous-répertoire pour la sauvegarde hebdomadaire
* **C_daily** : Sous-répertoire pour la sauvegarde mensuelle
* **C_notice** : Marqueur d’une ligne de log contenant un message d’information
* **C_warning** : Marqueur d’une ligne de log contenant un message d’alerte
* **C_error** : Marqueur d’une ligne de log contenant un message d’erreur

### Variables spécifiques à un projet

* **C_projectname** : Sous-répertoire pour la sauvegarde d’un projet
* **D_host** : hôte pour la connexion SSH
* **D_user** : login pour la connexion SSH
* **D_port** : port pour la connexion SSH
* **D_save** : si '1', sauvegarde les fichiers
* **D_srcdir** : Liste des répertoires distants à sauvegarder (requiert D_save=1)
* **D_rotd** : Rotation des logs pour les sauvegardes quotidiennes (sauvegardes des fichiers)
* **D_rotw** : Rotation des logs pour les sauvegardes hebdomadaires (sauvegardes des fichiers)
* **D_rotm** : Rotation des logs pour les sauvegardes mensuelles (sauvegardes des fichiers)
* **M_host** : hôte pour la connexion mysql
* **M_user** : login pour la connexion mysql
* **M_pass** : mot de passe pour la connexion mysql
* **M_port** : port pour la connexion mysql
* **M_save** : si '1', sauvegarde les BDD
* **M_bdd** : Liste des BDD distantes à sauvegarder (requiert M_save=1)
* **M_rotd** : Rotation des logs pour les sauvegardes quotidiennes (sauvegardes des BDD)
* **M_rotw** : Rotation des logs pour les sauvegardes hebdomadaires (sauvegardes des BDD)
* **M_rotm** : Rotation des logs pour les sauvegardes mensuelles (sauvegardes des BDD)

## Procédure pour ajouter un nouveau projet de sauvegarde
1. Copier le fichier model.conf dans le dossier available : `cp model.conf available/nom_client.conf`
2. Modifier le fichier nom_client.conf avec les paramètres associés
3. Activer la configuration du nouveau client en faisant un lien symbolique dans le dossier enabled : `ln -s /etc/backupclient/available/nom_client.conf /etc/backupclient/enabled/nom_client.conf`

_**Nota** : Pour s’assurer de la connexion SSH, ne pas oublier d’envoyer la clé du serveur de sauvgarde au serveur à sauvegarder via la commande `ssh-copy-id` et de s’assurer que le serveur de sauvegarde est autorisé à se connecter en SSH sur le serveur à sauvegarder (paramétrage iptables du serveur à sauvegarder)._

## Exemple d’arborescence du dossier de sauvegarde

```
/backupclient/
├── client1/
│   ├── data/
│   │   ├── daily.d/
│   │   │   ├── client1_2017-01-01_05h39.tar.gz
│   │   │   ├── client1_2017-01-02_05h39.diff.tar.gz
│   │   │   └── client1_2017-01-03_05h39.diff.tar.gz
│   │   ├── weekly.d/
│   │   │   ├── client1_2017-01-01_07h10.tar.gz
│   │   │   └── client1_2017-01-08_07h10.diff.tar.gz
│   │   └── monthly.d/
│   │       ├── client1_2017-01-05_08h00.tar.gz
│   │       ├── client1_2017-02-05_08h00.tar.gz
│   │       └── client1_2017-03-05_08h00.tar.gz
│   └── mysql/
│       ├── daily.d/
│       │   ├── client1_2017-01-01_05h38.tar.gz
│       │   ├── client1_2017-01-02_05h38.tar.gz
│       │   └── client1_2017-01-03_05h38.tar.gz
│       ├── weekly.d/
│       │   ├── client1_2017-01-01_07h08.tar.gz
│       │   └── client1_2017-01-08_07h07.tar.gz
│       └── monthly.d/
│           ├── client1_2017-01-05_07h30.tar.gz
│           ├── client1_2017-02-05_07h30.tar.gz
│           └── client1_2017-03-05_08h00.tar.gz
└── client2/
    ├── data/
      <etc.>
```