// ===========================================
// RAPPORT DES MODIFICATIONS EFFECTUES 
// ===========================================

 
// ===========================================
// 1. RESSOURCES AZURE
// ===========================================

➔ Modification de la Resource Group :
- Mise à jour de la Resource Group utilisée pour déployer l’infrastructure complète via Azure DevOps.
- Objectif : automatiser le déploiement dans Azure tout en respectant la structure imposée (4 lettres pour le nom du groupe).

➔ Modification de la Souscription Azure (Service de connexion Azure DevOps) :
- Ajout d'un Service Principal dans Azure DevOps pour permettre aux pipelines d’accéder à Azure.
- Attribution des rôles nécessaires au Service Principal :
  - Owner (propriétaire de la souscription)
  - Global Administrator
  - Key Vault Secrets Officer (droits sur Key Vault)
  - AcrPull (droits sur le registre Azure Container Registry)
- Lors de l'ajout du service de connexion Azure, la souscription est autorisée à enregistrer dynamiquement l'AppRegistration utilisée pour les applications déployées.

 
// ===========================================
// 2. MODIFICATIONS FICHIER PAR FICHIER
// ===========================================

➔ appsettings.json (dossier de chaque application à déployer) :
- Nettoyage du fichier pour supprimer tout hardcodage.
- La configuration est dorénavant chargée dynamiquement depuis Azure App Configuration et Azure Key Vault.
- Ajustement pour prendre en compte le nouveau endpoint configuré("https://emmaappconfiguration.azconfig.io") par l’infrastructure ARM lors du déploiement.

➔ worker_content/Program.cs :
- Ligne 37 :
  - Modification de la création du client pour Azure App Configuration.
  - Correction et mise à jour de la récupération du EventHubName depuis Azure App Configuration.
  - Objectif : éviter tout codage en dur du EventHubName ; il est désormais chargé depuis la configuration cloud.

➔ MVC/Program.cs :
- Ligne 134 :
  - Modification de l’injection du EventHubController provenant du BusinessLayer.
  - Correction de la syntaxe de récupération du EventHubName depuis Azure App Configuration.
  - Objectif : garantir que l'EventHubName est injecté dynamiquement sans hardcoding.

➔ azure-pipelines.yml (Pipeline  – Déploiement de l’infrastructure) :
- Modification :
  - Nom de la Resource Group (4 lettres ).
  - Utilisation du Service Principal pour authentification Azure (abonnement Azure configuré).
  - Valeur du Personal Access Token (PAT) mise à jour pour autoriser les étapes nécessitant l’accès aux ressources Azure DevOps.

➔ azure-pipelines-1.yml (Pipeline  - build and publisch Docker) :
- Similaire à azure-pipelines.yml :
  - Ajustements liés au Service Principal, à la Resource Group et au PAT pour assurer l'alignement avec l’environnement cible.

➔ azure-pipelines-2.yml (Pipeline - deploy ACI) :
- Alignement de la souscription, du Service Principal et de la Resource Group.
- Assure la continuité de la construction ou du déploiement d’éléments spécifiques.

➔ azure-pipelines-3.yml (Pipeline 3 – Déploiement de kube infrastructure) :
- Modifications principales :
  - Correction du nom de la Resource Group (4 lettres).
  - Mise à jour de la souscription (Service Principal correct).
  - Récupération dynamique dans les variables :
    - AppConfigEndpoint
    - KeyVaultName
    - SecretsFilter (pour filtrer les secrets du Key Vault pour Kubernetes)
- Utilisation des secrets Kubernetes créés dans cette pipeline pour configurer les workers API et services EventHub.

➔ deployKube/cluster.yml :
- Modification du fichier de déploiement Kubernetes :
  - Utilisation des secrets Kubernetes créés précédemment dans la pipeline 3.
  - Configuration de l’autoscaling :
    - HPA (Horizontal Pod Autoscaler) pour l'API.
    - KEDA (Kubernetes Event-driven Autoscaling) pour les workers.
- Objectif : permettre un scaling dynamique basé sur la charge ( messages EventHub) et non statique.

