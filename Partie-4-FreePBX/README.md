# Déploiement et Configuration de la Téléphonie (FreePBX & VoIP.ms)

> **Définition :** Mise en place d'un serveur de téléphonie IP centralisé (PBX) sous FreePBX interconnecté avec un fournisseur SIP externe (VoIP.ms), couplé à une gestion avancée des heures d'ouverture, des serveurs vocaux interactifs (SVI/IVR) et des postes clients logiciels (Zoiper 5).

## Fonctionnalités clés et Composants

- **Trunk SIP :** Connexion et authentification PJSIP sortante/entrante avec VoIP.ms.
- **SVI / IVR (Menu Interactif) :** Routage dynamique des appels par tonalités DTMF (touches 1 à 8).
- **Groupes de Sonnerie (Ring Groups) :** Stratégie de sonnerie simultanée (`ringall`) pour les équipes.
- **Gestion Horaire (Time Conditions) :** Automatisation du passage entre les heures d'ouverture et de fermeture.
- **Enregistrements Vocaux (System Recordings) :** Centralisation des messages d'accueil et d'absence.

---

## Architecture et Paramètres Réseau du Lab

- **Serveur PBX :** FreePBX (Hébergé sur VM VMware avec l'adresse IP statique `192.168.0.20`).
- **Fournisseur SIP Trunk :** VoIP.ms (Serveur régional `montreal1.voip.ms` sur le port UDP `5060`).
- **Clients / Softphones :** Zoiper 5 déployé sur les postes de travail virtuels clients (`PC-MTL-01`).
- **Topologie réseau spécifique :** Utilisation d'une double interface réseau sur les clients (Carte 1 sur `VMnet10` pour l'Active Directory, Carte 2 en mode *Bridged* pour isoler et garantir le trafic VoIP vers le PBX).

---

## Étapes de Configuration du Laboratoire

### 1. Configuration sur le Portail VoIP.ms
* **Création du sous-compte :** Génération d'un sous-compte SIP avec le protocole recommandé et le type de périphérique réglé sur *ATA, IP Phone or Softswitch*.
* **Paramètres de liaison :** Récupération de l'identifiant, du mot de passe sécurisé et du serveur cible (`montreal1.voip.ms`).

### 2. Création du Trunk PJSIP dans FreePBX
1. Navigation vers **Connectivity > Trunks > Add PJSIP Trunk**.
2. **Onglet Général :** Nommage du trunk (`VoIP.ms-Trunk`).
3. **Onglet pjsip Settings :**
   * *Authentication* : Configuré sur `Outbound`.
   * *Username & Secret* : Saisie des identifiants du sous-compte VoIP.ms.
   * *SIP Server* : `montreal1.voip.ms` sur le port `5060` avec le contexte `from-pstn`.
4. Validation par **Submit** et application des modifications via le bouton **Apply Config**.

### 3. Configuration des Routes (Inbound / Outbound)
* **Route Sortante (*Outbound Routes*) :** Création d'une règle de routage avec motifs de composition (`NXXNXXXXXX` / `1NXXNXXXXXX`) assignée au trunk VoIP.ms.
* **Route Entrante (*Inbound Routes*) :** Association du numéro DID entrant vers les conditions horaires (*Time Conditions*).

### 4. Enregistrements et Automatisation Horaire
* **System Recordings :** Téléversement et enregistrement du message d'accueil de l'entreprise (`Menu_Principal`) et du message de fermeture des bureaux.
* **Time Conditions :** Définition des plages horaires de travail (jours ouvrables et heures de bureau) pour aiguiller l'appel vers le SVI en journée ou vers le message d'absence en dehors des heures de service.

### 5. Configuration du SVI / Menu Interactif (IVR)
1. Création du menu `Menu_Principal` sous **Applications > IVR**.
2. Liaison du message d'accueil (*Announcement*) et configuration des entrées de touches (DTMF) :
   * **Touches 1 à 7 :** Redirection vers les départements spécifiques (*Direction*, *Finance*, *Marketing*, *RH*, *Service-client*, *Production*, *Informatique*).
   * **Touche 8 :** Redirection vers le groupe de sonnerie global des employés.

### 6. Création des Postes et du Groupe de Sonnerie (Ring Group 800)
* **Extensions individuelles :** Enregistrement et attribution des numéros pour l'ensemble des employés du bureau (ex. `1001`, `2030`, etc.).
* **Ring Group 800 (`Tous les postes`) :**
  * *Stratégie de sonnerie* : `ringall` (sonnerie simultanée).
  * *Temps de sonnerie* : 30 secondes.
  * *Announcement* : Réglé strictement sur **`None`** (pour éviter la boucle de ré-écoute du SVI lors de l'appel du groupe).

---

## Déploiement Client et Tests d'Appels (Zoiper 5)

1. **Installation logicielle :** Déploiement de **Zoiper 5** sur les postes clients de travail.
2. **Enregistrement SIP :** Configuration et association simultanée de deux clients de test avec leurs extensions respectives (ex: `1001` et `2030`), validées par le statut de connexion au vert (**Registered**).
3. **Tests d'appels :** Réalisation de simulations d'appels entrants à travers le SVI et validation de la sonnerie simultanée sur les postes décrochés.

---

## Dépannage et Résolution des Incidents (Troubleshooting)

### 1. Isolation Réseau (`VMnet10`)
* **Symptôme :** Impossibilité pour les clients Active Directory sur `VMnet10` de joindre le serveur PBX (`192.168.0.20`), entraînant un échec d'enregistrement SIP (*Timeout 408*).
* **Résolution :** Ajout d'une deuxième carte réseau en mode *Pont (Bridged)* sur le poste client (`PC-MTL-01`), permettant d'isoler le trafic du domaine tout en routant la téléphonie vers le réseau principal.

### 2. Conflit de Numérotation du SVI
* **Symptôme :** L'affectation de la touche `8` pointant vers un groupe nommé `8` provoquait un saut direct de l'appel sans sonnerie à cause d'un conflit avec les codes de service internes d'Asterisk.
* **Résolution :** Migration du groupe vers un format à trois chiffres normalisé (`800`), aligné avec l'architecture des autres groupes de l'entreprise (`100`, `200`, etc.).

### 3. Filtrage Pare-Feu et Appels Externes (Cellulaire Personnel)
* **Symptôme :** Les appels émis depuis un téléphone mobile personnel vers le DID ne parvenaient pas jusqu'au menu IVR.
* **Résolution :** Analyse et ouverture des flux sur le pare-feu (règles de sécurité et autorisations de routage des paquets SIP/RTP) pour autoriser le trafic externe vers l'IP du serveur FreePBX.

---

## Résultats Attendus de l'Infrastructure

![Architecture et Interfaces de Test](Images/VoIP-Architecture-Test.png)
