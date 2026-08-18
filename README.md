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

## 4. Automatisation du Mappage Réseau (GPO)
Mise en place et liaison de la stratégie GPO_LecteursReseaux sur l'OU Citadelle_Solutions :

* Lecteur G: (Public) : Mappage automatique du dossier \\DC-MTL\DATA$\Public pour le groupe GG_TousLesEmployes.
* Lecteur S: (Service) : Mappage dynamique du dossier de département correspondant à l'utilisateur (\\DC-MTL\DATA$\<NomDuService>) grâce au Ciblage par élément (Item-Level Targeting) basé sur l'appartenance aux groupes GG_*.
* Lecteur P: (Dossier Personnel) : Connexion automatique d'un répertoire individuel sécurisé pour chaque utilisateur lors de l'ouverture de session.
