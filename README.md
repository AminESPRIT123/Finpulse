# FINPULSE

**Voir venir, décider plus tôt.**

FINPULSE est une application destinée aux petites et moyennes
entreprises (PME). Elle les aide à centraliser le suivi de leurs
factures, paiements et mouvements de trésorerie, à prévoir leur
situation financière et à détecter les risques avant qu'ils ne
deviennent critiques.

## 1. Présentation du projet

Les PME suivent souvent leurs factures, leurs règlements et leurs
dépenses à travers plusieurs fichiers ou outils. Cette dispersion rend
le suivi financier plus difficile et peut retarder la détection des
problèmes de trésorerie.

FINPULSE propose une plateforme centralisée qui permet de : - gérer les
informations de l'entreprise, ses clients et ses fournisseurs ; - créer
et suivre les factures ainsi que les paiements ; - suivre les mouvements
et le solde de trésorerie ; - établir des prévisions de trésorerie à 30,
60 et 90 jours ; - détecter des risques financiers à partir de règles
métier ; - informer les utilisateurs grâce à un système de
notifications.

## 2. Objectif du MVP et périmètre

Le MVP couvre un parcours fonctionnel complet :

**Facture → Prévision de trésorerie → Détection d'un risque →
Notification.**

La première version utilise des données fictives pour tester et
démontrer le fonctionnement de la solution.

Les éléments suivants ne font pas partie du périmètre du MVP : -
connexion à de véritables comptes bancaires ; - paiement en ligne ; -
octroi de crédits ; - détection fondée sur le Machine Learning ; -
comptabilité légale ; - gestion multi-devises.

Les prévisions sont des projections calculées à partir des opérations
connues. Elles ne constituent pas une garantie du solde réellement
disponible dans le futur.

## 3. Acteurs

-   **ADMIN (administrateur) :** configure les informations et les
    paramètres de l'entreprise et consulte les informations auxquelles
    son rôle lui donne accès.
-   **COLLABORATEUR :** utilise les fonctionnalités opérationnelles de
    l'application, notamment le suivi des factures, des mouvements et le
    traitement des alertes, selon les autorisations définies.

## 4. Architecture et découpage en microservices

L'architecture suit une approche microservices inspirée du Domain-Driven
Design (DDD). Chaque capacité métier est isolée dans un contexte et
possède sa propre base de données.

Le projet identifie six contextes : cinq microservices développés par
l'équipe et un service d'identité et d'accès fourni par Keycloak.

  -------------------------------------------------------------------------------------------
  Service           Responsabilité                  Port Technologie   Base de données
  ----------------- ------------------ ----------------- ------------- ----------------------
  **MS1 --          Entreprise,                     8081 Spring Boot   MySQL
  Entreprise &      clients,                                           (`entreprise_db`)
  tiers**           fournisseurs et                                    
                    paramètres                                         

  **MS2 --          Cycle de vie des                8082 Spring Boot   MySQL
  Facturation**     factures et des                                    (`facturation_db`)
                    règlements                                         

  **MS3 --          Mouvements, solde               8083 Spring Boot   MySQL
  Trésorerie**      et prévisions de                                   (`tresorerie_db`)
                    trésorerie                                         

  **MS4 --          Règles de                       8084 Spring Boot   MySQL (`risques_db`)
  Risques**         détection et suivi                                 
                    des alertes                                        

  **MS5 --          Historique et                   8085 NestJS ou     MySQL
  Notifications**   lecture des                          FastAPI\*     (`notifications_db`)
                    notifications                                      

  **Identité &      Authentification                 --- Keycloak      Géré par Keycloak
  accès**           et gestion des                                     
                    rôles                                              
  -------------------------------------------------------------------------------------------

*La technologie de MS5 reste à confirmer selon le choix final de
l'équipe et les consignes de l'encadrement.*

### Rôle des microservices

-   **MS1 -- Entreprise & tiers :** conserve les informations de
    référence de l'entreprise, de ses clients, fournisseurs et
    paramètres.
-   **MS2 -- Facturation :** gère les factures et les règlements
    associés.
-   **MS3 -- Trésorerie :** suit les encaissements et décaissements et
    calcule les prévisions.
-   **MS4 -- Risques :** applique des règles métier explicables pour
    détecter notamment les retards de paiement, les tensions de
    trésorerie, la concentration des créances sur un client et les
    sorties élevées.
-   **MS5 -- Notifications :** conserve les notifications destinées aux
    utilisateurs et permet leur consultation et leur lecture.
-   **Keycloak :** gère l'authentification, les utilisateurs et les
    rôles `ADMIN` et `COLLABORATEUR`.

## 5. Communication entre les services

L'application utilise deux principaux modes de communication.

### Communication synchrone

Les appels REST entre microservices sont réalisés notamment avec
**OpenFeign**. Le service appelant attend la réponse du service
destinataire.

Exemples : - MS2 consulte MS1 pour vérifier les informations d'un client
ou partenaire. - MS3 récupère auprès de MS1 les informations de
l'entreprise nécessaires à ses calculs. - MS4 récupère auprès de MS1 les
paramètres financiers et auprès de MS2 les factures échues.

### Communication asynchrone

**RabbitMQ** transporte les événements publiés par un service et
consommés par un autre.

Exemples : - MS2 publie `InvoiceCreated` et `PaymentRecorded`, consommés
par MS3. - MS3 publie `TreasuryForecastUpdated`, consommé par MS4. - MS4
publie `RiskAlertCreated`, consommé par MS5.

## 6. Technologies et infrastructure

-   **Frontend :** Angular et TypeScript.
-   **Backend :** Spring Boot et Java pour MS1 à MS4.
-   **MS5 :** NestJS ou FastAPI, choix à confirmer.
-   **Bases de données :** MySQL, avec une base indépendante par
    microservice.
-   **API Gateway :** Spring Cloud Gateway, point d'entrée des requêtes
    vers les services.
-   **Découverte de services :** Eureka.
-   **Configuration centralisée :** Spring Cloud Config Server.
-   **Authentification et autorisation :** Keycloak et jetons JWT.
-   **Messagerie asynchrone :** RabbitMQ.
-   **Conteneurisation :** Docker et Docker Compose.

## 7. Répartition des membres

  -----------------------------------------------------------------------
  Membre                              Responsabilités principales
  ----------------------------------- -----------------------------------
  **Amina Zhioua**                    MS1 -- Entreprise & tiers ;
                                      Gateway, Eureka et Config Server

  **Imen Charfi**                     MS2 -- Facturation ; Docker
                                      Compose, RabbitMQ et
                                      schémas/utilisateurs MySQL

  **Med Amin Ben Salem**              MS3 -- Trésorerie ; configuration
                                      du realm Keycloak, rôles, jetons et
                                      sécurité des services

  **Molka Boukoum**                   MS4 -- Risques ; tableau de bord
                                      frontend et tests de bout en bout

  **Toute l'équipe**                  MS5 -- Notifications, développé
                                      collectivement
  -----------------------------------------------------------------------

Chaque membre contribue également à l'interface Angular de son
microservice. Le service Notifications est un travail collectif ; sa
présentation est assurée par un membre désigné par l'équipe.

## 8. Principes d'architecture

-   Chaque microservice est responsable de ses propres données.
-   Les services ne consultent pas directement les tables des bases de
    données des autres services.
-   Les échanges de données passent par des API ou des événements.
-   L'API Gateway constitue le point d'entrée de l'application.
-   Les accès sont contrôlés à partir du JWT et des rôles.
-   Les événements doivent être traités de manière à éviter les
    doublons.

## 9. État du projet

Ce dépôt contient le travail du projet **FINPULSE** réalisé dans le
cadre du projet académique de l'équipe. Les fonctionnalités, choix
techniques et éléments de configuration peuvent évoluer au cours du
développement.

------------------------------------------------------------------------

**FINPULSE --- Voir venir, décider plus tôt.**
