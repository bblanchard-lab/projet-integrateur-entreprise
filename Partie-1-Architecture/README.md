# Partie 1 : Architecture AD DS, Serveur de Fichiers & Sécurité NTFS

Cette première étape du projet consiste à poser les bases solides de l'infrastructure réseau de **Citadelle Solutions**. L'objectif est de structurer un annuaire Active Directory multi-sites (`cita.com`), de préparer le serveur de fichiers centralisé sur le contrôleur de domaine de Montréal (`DC-MTL`) et de sécuriser rigoureusement les accès aux données de l'entreprise.

---

## 1. Sécurisation de l'Administration & Services de Base

* **Gestion des comptes privilèges :** * Renommage du compte `Administrateur` intégré du domaine et application d'une restriction GPO pour interdire son ouverture de session à distance sur l'ensemble des postes clients.

  * Création d'un compte d'administration nominatif (membre de `Domain Admins`) respectant la convention de nommage globale de l'entreprise pour effectuer toutes les tâches d'administration quotidiennes.

* **Services DNS :**

  * Configuration des zones DNS intégrées à Active Directory avec réplication sur tout le domaine.
  * Activation exclusive des **mises à jour dynamiques sécurisées** (blocage strict de toute mise à jour non sécurisée).

* **Service DHCP :**

  * Déploiement et autorisation du rôle DHCP sur le contrôleur de domaine principal à Montréal (`DC-MTL`).
  * Plage distribuée : `192.168.0.10` à `192.168.0.254` (Masque `/24`).
  * Réservation d'adresses statiques : `192.168.0.1` à `192.168.0.9`.
  * Durée du bail définie à **5 jours**.

---

## 2. Structure Active Directory (AD DS) & Modèle AGDLP

Pour assurer une gestion fluide et évolutive des ressources, l'annuaire a été organisé par bureau sous une Unité Organisationnelle (OU) principale :

```

[cita.com/](https://cita.com/)
└── Citadelle_Solutions/
    ├── Montreal/
    │   ├── Groupes/        --> Groupes de sécurité globaux (GG_*)
    │   ├── Ordinateurs/    --> Postes de travail de Montréal
    │   └── Utilisateurs/   --> Comptes du personnel de Montréal
    └── Quebec/
        ├── Ordinateurs/    --> Postes de travail de Québec
        └── Utilisateurs/   --> Comptes du personnel de Québec

```

Groupes de sécurité globaux créés :

Départements : GG_Direction, GG_Finance, GG_Marketing, GG_RH, GG_ServiceClients, GG_Production, GG_Informatique.
Organisationnel : GG_TousLesEmployes (regroupant l'ensemble du personnel de Montréal et de Québec).

## 3. Stockage, Partage et Sécurité NTFS

Préparation du volume de données :

* Ajout d'un second disque dur virtuel dédié aux données sur le serveur de fichiers.
* Modification de la lettre du lecteur DVD vers E: afin de réassigner la lettre D: au nouveau volume (initialisé en GPT, formaté en NTFS et nommé DATA).
* Création du dossier racine D:\DATA et configuration du partage réseau masqué DATA$ (\\DC-MTL\DATA$).
* Autorisation au niveau du partage : Contrôle total pour Tout le monde (les accès réels étant restreints par la sécurité NTFS).

Matrice des autorisations NTFS sur les répertoires :

L'héritage des autorisations a été rompu sur tous les sous-dossiers de D:\DATA\ pour appliquer la politique d'accès suivante :
* Marketing : Droits en modification pour GG_Marketing, lecture/écriture pour GG_Production et administration pour GG_Informatique.
* Finance : Droits en modification pour GG_Finance, lecture/écriture pour GG_Direction et administration pour GG_Informatique.
* Direction : Droits en modification pour GG_Direction et administration pour GG_Informatique.
* ServiceClients : Droits en modification pour GG_ServiceClients et GG_Direction, lecture/écriture pour GG_RH et administration pour GG_Informatique.
* ServiceClients\VIP : Accès restreint spécifique (droit d'administration informatique exclu).
* RH : Droits en modification pour GG_RH, lecture/écriture pour GG_Direction et administration pour GG_Informatique.
* RH\Confidentiel : Accès restreint exclusivement à la Direction et aux responsables RH (droit d'administration informatique exclu).
* Production : Droits en modification pour GG_Production et administration pour GG_Informatique.
* Public : Droits en modification pour l'ensemble des employés via GG_TousLesEmployes et administration pour GG_Informatique.


## 4. Automatisation du Mappage Réseau par GPO (Lettres officielles du document)

Mise en place et liaison de la stratégie GPO_LecteursReseaux sur l'OU Citadelle_Solutions pour automatiser la connexion des lecteurs à l'ouverture de session:  
* $X:= DATA\Marketing
* $Y:= DATA\Finance
* $Z:= DATA\Direction
* $M:= DATA\ServiceClients
* $N:= DATA\RH
* $H:= DATA\Production
* $G:= DATA\Public   
* $P:= Dossier Personnel (Connecté automatiquement pour chaque utilisateur)

## 5. Configuration des Stations de Travail, Sécurité & Hardening

Pour répondre aux exigences de la direction, l'ensemble des postes de travail clients est soumis à des politiques de groupe (GPO) visant à restreindre les privilèges des utilisateurs standards (à l'exception du technicien TI) et à appliquer un durcissement (hardening) strict.

### 5.1. Configuration du Bureau Standard (Droits Restreints)
Création et liaison d'une GPO dédiée appliquée aux Unités Organisationnelles des utilisateurs afin de verrouiller l'environnement de travail :
* **Menu Exécuter :** Suppression du raccourci dans le menu Démarrer.
* **Outils système :** Blocage de l'accès à l'Invite de commandes (CMD), au Registre (`regedit`) et au Gestionnaire de tâches via les paramètres de stratégie d'administration.
* **Corbeille :** Masquage et interdiction d'accès à l'onglet des propriétés de la Corbeille.
* **Console PowerShell :** Désactivation de l'exécutable PowerShell pour les comptes non-administrateurs afin de limiter la surface d'attaque en cas d'exécution de scripts malveillants.

---

### 5.2. Stratégie de Sécurité des Comptes & Durcissement (GPO)
Application des règles de sécurité sur la stratégie de domaine par défaut (*Default Domain Policy*) et via des GPOs ciblées :

* **Politique de mots de passe :**
  * **Longueur minimale :** 12 caractères.
  * **Exigences de complexité :** Activées (majuscules, minuscules, chiffres, caractères spéciaux).
  * **Historique :** Conservation des 5 derniers mots de passe pour empêcher la réutilisation immédiate.
  * **Expiration :** Renouvellement forcé tous les 180 jours (6 mois).
  * **Dictionnaire :** Blocage des mots de passe courants/simplistes via le filtrage natif AD.

* **Verrouillage des comptes & Session :**
  * **Seuil de verrouillage :** Compte verrouillé après 3 tentatives infructueuses.
  * **Durée de verrouillage :** Réactivation possible après 15 minutes.
  * **Mise en veille :** Verrouillage automatique de la session de travail après 15 minutes d'inactivité.

* **Sécurité du système & Confidentialité :**
  * **Comptes intégrés :** Désactivation systématique du compte `Invité` sur toutes les machines.
  * **Supports amovibles :** Désactivation complète de l'exécution automatique (*Autorun/Autoplay*) sur les clés USB et lecteurs amovibles.
  * **Télémétrie :** Blocage de l'envoi de données de diagnostic vers Microsoft.
  * **Confidentialité :** Désactivation de l'historique d'activités et du suivi utilisateur.

---

### 6. Chiffrement du Stockage (BitLocker)
Déploiement d'une politique de chiffrement centralisée pour protéger les données au repos sur les postes de travail :
* **Chiffrement des volumes :** Activation obligatoire de BitLocker sur les disques système des stations.
* **Sauvegarde AD DS :** Configuration de la GPO pour forcer le stockage automatique des clés de récupération BitLocker directement dans les objets ordinateurs de l'Active Directory (*Active Directory Domain Services*).
