# Microsoft Endpoint Configuration Manager (MECM)

> **Définition :** Microsoft Endpoint Configuration Manager (anciennement SCCM) administre de manière centralisée les parcs informatiques (PC, serveurs, mobiles).

## Fonctionnalités clés

- **Déploiement :** Applications et systèmes d'exploitation (OS).
- **Mises à jour :** Gestion via WSUS / Cloud.
- **Sécurité & Conformité :** Protection des terminaux, suivi de la conformité.
- **Gestion & Inventaire :** Inventaire matériel/logiciel, gestion de l'alimentation, et rapports (SSRS).

---

## Architecture et Hiérarchie de Site

- **Site principal autonome :** Recommandé pour les déploiements de petite/moyenne taille (jusqu'à 150 000 clients). Regroupe tous les rôles sur un seul serveur ou les répartit.
- **Site secondaire :** Utilisé pour les sites distants (> 500 clients) à bande passante limitée. Géré depuis le site principal, prend en charge 15 000 clients.
- **Site d'administration centrale (CAS) :** Sommet de la hiérarchie pour les très grands déploiements (jusqu'à 25 sites principaux). Ne gère pas directement les appareils clients.

---

## Prérequis Logiciels et Matériels

### Système & Base de données
- **OS :** Windows Server 2022
- **BDD :** SQL Server 2022 *(Collation obligatoire : `SQL_Latin1_General_CP1_CI_AS`)*

### Outils d'installation
- Windows ADK *(v10.1.26100.2454 ou supérieure)* avec extension **Windows PE**
- Driver ODBC
- SQL Server Reporting Services (SSRS)
- SQL Server Management Studio (SSMS 21)

### Matériel requis (Site principal autonome - BDD locale)
- **CPU :** 16 cœurs
- **RAM :** 96 Go *(allocation de 80 % réservée à SQL Server)*

---

## Étapes de Configuration du Laboratoire

### 1. Préparation d'Active Directory (sur le DC)
* **Conteneur System Management :** Créé dans ADSI Edit sous `CN=System`.
* **Permissions :** Attribution du *Full Control* au compte ordinateur du serveur MECM (`MECM$`) appliqué à *« Cet objet et tous les objets enfants »*.
* **Comptes et Groupes :**
  * Création du compte de service `sqlservice@cita.com` dans l'OU des comptes de service.
  * Création des groupes de sécurité `MECMadmins` et `SQLAdmins`.
  * Attribution du droit local *Log on as a service* au compte `sqlservice` sur le serveur MECM.

### 2. Extension du Schéma Active Directory
1. Exécution de l'outil `extadsch.exe` en administrateur.
2. Validation dans le fichier journal `C:\ExtADSch.log` *(confirmer le message : `Successfully extended the Active Directory schema`)*.

### 3. Installation des Composants SQL & Rapports
* **SQL Server 2022 :** Installation du moteur de BDD avec le compte `sqlservice`, la collation spécifique et l'attribution des droits d'administration aux groupes dédiés.
* **SSRS & SSMS :** Installation et configuration de SQL Server Reporting Services, puis installation de SSMS 21 pour la gestion.
* **Ajustement mémoire :** Limiter la mémoire maximale de SQL Server dans les propriétés de l'instance sous SSMS *(ex: 2048 Mo en environnement de test)*.

### 4. Dépendances Windows Server & Installation de MECM
* **Rôles et fonctionnalités :** Installation de WSUS (avec connectivité SQL), BITS, et Remote Differential Compression (RDC).
* **Déploiement MECM :** Exécution du programme d'installation (`splash.exe`), choix du site principal autonome, acceptation des licences, téléchargement des fichiers requis, et assignation du code de site *(ex: `SA1`)*.
