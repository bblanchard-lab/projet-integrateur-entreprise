# Projet Intégrateur - Déploiement & Sécurisation d'Infrastructure d'Entreprise

## Présentation du projet
Ce projet met en place l'infrastructure informatique complète de l'entreprise fictive **Citadelle Solutions** (domaine `cita.com`), répartie sur deux sites principaux (Montréal et Québec). 

L'objectif est d'assurer un environnement centralisé, hautement sécurisé et entièrement géré par des stratégies de groupe (GPO) et des outils d'administration avancés.

---

## Architecture globale & Vue d'ensemble

* **Domaine Active Directory :** `cita.com`
* **Serveur principal :** `DC-MTL` (Windows Server 2022)
* **Services centralisés :** AD DS, DNS sécurisé, DHCP, Serveur de fichiers SMB, GPO de durcissement.
* **Gestion des postes :** Chiffrement BitLocker, restriction des privilèges et automatisation.

---

## Structure du répertoire

Voici le découpage détaillé du projet. Cliquez sur les sous-dossiers pour accéder au rapport technique complet de chaque phase :

*  **[Partie 1 : Architecture AD DS, Serveur de Fichiers & Sécurité NTFS](./Partie-1-Architecture/)**
  * Structure des OUs et modèle AGDLP (Montréal & Québec).
  * Configuration du serveur de fichiers (`DATA$`) et matrice de droits NTFS.
  * Mappage réseau automatique par GPO.
  * Durcissement des postes clients (GPO, restrictions système, politiques de sécurité).
  * Chiffrement des stations via BitLocker avec sauvegarde des clés dans l'AD.

*  **[Partie 2 : Gestion des Systèmes & Déploiement MECM / SCCM](./Partie-2-MECM/)**
  * *(Résumé rapide des gros points de ta partie 2 ici)*.

---

## Technologies & Compétences démontrées
* **Administration système :** Windows Server 2022, Active Directory DS, Group Policy (GPO).
* **Réseau de base :** DNS (Updates sécurisées), DHCP (Réservations & Baux).
* **Sécurité & Hardening :** Modèle AGDLP, permissions NTFS avancées, BitLocker, restriction PowerShell/CMD.
