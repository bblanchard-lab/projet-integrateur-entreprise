# Configuration et Déploiement de SharePoint 2019 : Architecture des Collections de Sites par Département

Ce dépôt contient la documentation technique complète et les procédures d'implémentation réalisées pour la **Partie 3** du laboratoire d'infrastructure Windows Server et SharePoint Server 2019.

---

## 📋 Table des matières
1. [Présentation de l'Architecture](#1-présentation-de-larchitecture)
2. [Matrice des Collections de Sites et Administrateurs](#2-matrice-des-collections-de-sites-et-administrateurs)
3. [Procédure Technique : Déploiement d'une Collection de Sites](#3-procédure-technique--déploiement-dune-collection-de-sites)
4. [Gestion des Rôles et Modèle de Sécurité](#4-gestion-des-rôles-et-modèle-de-sécurité)
5. [Résolution d'Incidents et Dépannage (Troubleshooting)](#5-résolution-dincidents-et-dépannage-troubleshooting)
6. [Validation et Tests d'Accès Clients](#6-validation-et-tests-daccès-clients)

---

## 1. Présentation de l'Architecture

L'objectif principal de cette phase est la mise en place d'une architecture modulaire, sécurisée et étanche de collections de sites SharePoint 2019 au sein du domaine Active Directory `CITA.com`. 

Chaque service organisationnel (incluant la **Direction**) bénéficie d'une collection de sites dédiée, garantissant l'isolation logique des données et l'impartition stricte de la gestion des contenus aux responsables métiers.

### Principes directeurs de l'architecture :
* **Isolation stricte des services :** Chaque département dispose de sa propre collection de sites sous la structure d'URL `/sites/`.
* **Principe du moindre privilège & Administration unique :** Chaque collection de sites est administrée par **un seul et unique employé désigné** (Administrateur principal). Le champ d'administrateur secondaire est maintenu strictement vide.
* **Découplage des comptes de service :** Les comptes de service Active Directory (ex: `CITA\webapp`, `CITA\sp-farm`) ne sont pas utilisés pour les activités de consultation ou de gestion quotidienne.

---

## 2. Matrice des Collections de Sites et Administrateurs

| Département / Service | URL de la Collection | Modèle de site | Administrateur Principal (`CITA\...`) | Administrateur Secondaire |
| :--- | :--- | :--- | :--- | :--- |
| **Direction** | `http://team.cita.com/sites/direction` | Site d'équipe | *(Employé Direction)* | *(Aucun)* |
| **Informatique** | `http://team.cita.com/sites/informatique` | Site d me | `cbenson` (Charles Benson) | *(Aucun)* |
| **Ressources Humaines** | `http://team.cita.com/sites/rh` | Site d'équipe | *(Employé RH)* | *(Aucun)* |
| **Finance** | `http://team.cita.com/sites/finance` | Site d'équipe | *(Employé Finance)* | *(Aucun)* |

---

## 3. Procédure Technique : Déploiement d'une Collection de Sites

> **Exemple de référence :** La procédure ci-dessous illustre l'implémentation pour le département **Informatique** avec l'administrateur `cbenson`. Cette méthode exacte a été appliquée pour l'ensemble des autres services.

### Étape 1 : Connexion à l'Administration Centrale
1. Se connecter sur la machine virtuelle serveur **SharePoint** (`192.168.0.11`) avec le compte administrateur du domaine (`CITA\admin-domaine`).
2. Lancer l'**Administration centrale de SharePoint 2019**.
3. Naviguer vers **Gestion des applications** (*Application Management*) > **Collections de sites** (*Site Collections*) > **Créer des collections de sites** (*Create site collections*).

### Étape 2 : Configuration des identifiants et de l'URL
* **Application Web :** S'assurer que l'application `http://team.cita.com` est sélectionnée.
* **Titre :** `Service Informatique`
* **Description :** `Espace de collaboration et ressources du service Informatique.`
* **Adresse URL :** Sélectionner le chemin d'inclusion `/sites/` et saisir `informatique`.
  * *URL complète :* `http://team.cita.com/sites/informatique`

### Étape 3 : Sélection du modèle et désignation de l'administrateur
1. **Sélection du modèle :** Sous l'onglet *Collaboration*, choisir **Site d'équipe**.
2. **Administrateur principal :** Saisir le compte `CITA\cbenson` et valider via le vérificateur de noms Active Directory.
3. **Administrateur secondaire :** Laisser le champ **vide** afin de respecter la contrainte d'un administrateur unique par service.
4. Cliquer sur **OK** et attendre la création de la collection de sites et l'instanciation de la base de données.

---

## 4. Gestion des Rôles et Modèle de Sécurité

La gestion des autorisations est déléguée à l'administrateur principal de chaque site, libérant l'équipe d'infrastructure réseau des tâches de gestion quotidienne des contenus.

```
                         [ Propriétaire du Site ]
                   (Ex: CITA\cbenson - Contrôle Total)
                                 │
         ┌───────────────────────┴───────────────────────┐
         ▼                                               ▼
[ Membres du Site ]                            [ Visiteurs du Site ]
(Collègues du département)                      (Utilisateurs hors service)
• Niveau : Modification / Écriture             • Niveau : Lecture seule
• Création & édition de documents              • Consultation documentaire uniquement
```

* **Propriétaires du site (*Control Total*) :** Uniquement l'employé responsable désigné lors de la création de la collection.
* **Membres du site (*Modification*) :** Employés rattachés au département pour la production documentaire.
* **Visiteurs du site (*Lecture seule*) :** Utilisateurs externes au service nécessitant un accès de consultation (ex: `CITA\rsmith`).

---

## 5. Résolution d'Incidents et Dépannage (Troubleshooting)

Durant le déploiement et la validation de l'infrastructure, plusieurs incidents techniques ont été identifiés et résolus :

### Incident 1 : Timeouts réseau (`ERR_CONNECTION_TIMED_OUT`)
* **Symptôme :** Les postes clients (ex: `PC-MTL-01`) ne parviennent pas à joindre `http://team.cita.com`.
* **Cause :** Blocage des requêtes HTTP (Port 80) entrantes par le pare-feu Windows du serveur SharePoint.
* **Solution :** Ajout d'une règle d'autorisation explicite dans PowerShell sur le serveur SharePoint :
  ```powershell
  New-NetFirewallRule -DisplayName "SharePoint HTTP Port 80 Inbound" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow -Profile Any
  ```

### Incident 2 : Réponse brute `HTTP/1.1 200 OK` dans le navigateur
* **Symptôme :** L'accès à une nouvelle collection de sites (ex: `/sites/rh`) renvoie un texte brut représentant les en-têtes HTTP au lieu de la page web.
* **Cause :** Saisie d'une URL incomplète ou délai de recyclage de l'Application Pool IIS suite à la création consécutive de plusieurs collections de sites.
* **Solution :**
  1. S'assurer d'appeler l'URL exacte ou cibler la page d'accueil (`/sites/rh/SitePages/Home.aspx`).
  2. Forcer le redémarrage des services IIS pour rafraîchir les pools d'applications :
     ```cmd
     iisreset
     ```

### Incident 3 : Configuration des liaisons IIS (Bindings)
* **Configuration appliquée dans le Gestionnaire IIS :**
  * **Type :** `http`
  * **Adresse IP :** `Toutes non attribuées` (*All Unassigned*)
  * **Port :** `80`
  * **Nom d'hôte :** `team.cita.com`

---

## 6. Validation et Tests d'Accès Clients

Les vérifications ont été réalisées depuis la machine virtuelle client **PC-MTL-01** connectée au domaine `CITA.com` :

1. **Test de connectivité et résolution DNS :**
   ```cmd
   ping team.cita.com
   ```
   *Résultat :* Résolution DNS valide vers l'adresse IP `192.168.0.11` avec 0% de perte de paquets.

2. **Validation de l'authentification et du contrôle d'accès :**
   * **Connexion en tant que `CITA\cbenson` :** Accès complet d'administration sur `http://team.cita.com/sites/informatique`.
   * **Connexion en tant que `CITA\rsmith` :** Accès restreint en lecture seule sur `http://team.cita.com/sites/portail`.

