// ===========================================
// README – DOCUMENTATION COMPLÈTE DES ÉTAPES SUIVIES
// ===========================================


// ===========================================
// 1. CRÉATION DES BASES CLOUD POUR L’ITÉRATION 7
// ===========================================

➔ Mise en place des ressources fondamentales dans Azure :
- Création d'un Resource Group respectant le standard (4 lettres).
- Ajout d'un Service Principal pour permettre aux pipelines Azure DevOps d’interagir avec Azure.
- Attribution des rôles au Service Principal :
  - Owner
  - Global Administrator
  - Key Vault Secrets Officer
  - AcrPull

➔ Objectif :
- Préparer une base stable pour accueillir toutes les ressources cloud nécessaires au projet.
- Sécuriser l’accès aux ressources sans codage en dur (via Azure App Configuration et Key Vault).


// ===========================================
// 2. CRÉATION DU REGISTRE DE CONTENEURS
// ===========================================

➔ Actions réalisées :
- Déploiement d’un Azure Container Registry sécurisé pour héberger les images de conteneurs.
- Suppression de tous les secrets et informations sensibles des images.
- Utilisation de variables d’environnement dynamiques injectées au moment du déploiement, via Kubernetes Secrets, App Configuration et Key Vault.

➔ Objectif :
- Sécuriser totalement les environnements de build et de run.
- Se conformer au principe : aucune donnée sensible codée en dur.


// ===========================================
// 3. COMPILATION ET DÉPLOIEMENT DES CONTENEURS
// ===========================================

➔ Actions réalisées :
- Compilation du projet pour générer les images Docker pour chaque composant (API, MVC, Worker_Content, Worker_Image, Worker_DB).
- Publication automatique des images dans Azure Container Registry via les pipelines Azure DevOps (`azure-pipelines.yml`, `azure-pipelines-1.yml`, `azure-pipelines-2.yml`).

➔ Objectif :
- Centraliser et sécuriser les images conteneurisées.
- Assurer une traçabilité complète du cycle de vie de chaque image.


// ===========================================
// 4. CRÉATION DES INFRASTRUCTURES CLOUD
// ===========================================

➔ Actions réalisées :
- Déploiement via ARM Templates de toutes les ressources nécessaires :
  - Azure Kubernetes Service (AKS)
  - Azure Key Vault
  - Azure App Configuration
  - Azure Monitor pour la surveillance
  - Event Hub pour les workers
  - Service Bus pour les queues de messages

➔ Objectif :
- Préparer une plateforme cloud scalable et sécurisée pour accueillir les 5 conteneurs.


// ===========================================
// 5. MISE EN PLACE DES MÉCANISMES D’ORCHESTRATION
// ===========================================

➔ Configuration mise en œuvre :
- **Autoscaling :**
  - API : Autoscalable en fonction de la charge CPU sur les pods (HPA).
  - MVC : Autoscalable en fonction de la charge CPU (HPA).
  - Worker_Content : Autoscalable en fonction du nombre de messages dans Service Bus (KEDA).
  - Worker_Image : Autoscalable en fonction du nombre de messages dans Service Bus (KEDA).
  - Worker_DB : Autoscalable en fonction du nombre d'événements dans Event Hub (KEDA).

- **Surveillance :**
  - Mise en place de l’Azure Monitor sur le cluster AKS pour superviser :
    - Les métriques d’utilisation
    - Les logs d'application
    - Les alertes personnalisées en cas de seuil critique

➔ Objectif :
- Assurer la haute disponibilité et la résilience automatique de toutes les applications.
- Avoir une visibilité temps réel sur la santé des ressources.


// ===========================================
// 6. DÉPLOIEMENT DES PACKAGES SUR L’INFRASTRUCTURE
// ===========================================

➔ Actions réalisées :
- Intégration et déploiement automatisé des conteneurs sur Kubernetes via les pipelines Azure DevOps (`azure-pipelines-3.yml`).
- Création et utilisation des Kubernetes Secrets pour injecter dynamiquement :
  - AppConfig endpoints
  - Key Vault secrets
  - Event Hub names

- Configuration du cluster Kubernetes pour que :
  - Les pods utilisent des valeurs dynamiques sans hardcodage.
  - L'orchestration se fasse via HPA (Horizontal Pod Autoscaler) et KEDA (Kubernetes Event-driven Autoscaler).

➔ Objectif :
- Déployer les applications de manière sécurisée, fiable, et automatisée.


// ===========================================
// 7. TESTS DE FONCTIONNEMENT
// ===========================================

➔ Tests réalisés :
- **Tests de Scalabilité** :
  - Vérification que les pods API, MVC, et les Workers scalent automatiquement en fonction de la charge générée.

- **Tests d’intégrité et de performance** :
  - Validation que toutes les applications fonctionnent correctement après déploiement.
  - Vérification de la communication entre API, MVC, et Workers via les ressources Event Hub et Service Bus.

- **Tests de sécurité** :
  - Confirmation qu’aucune information sensible n’est exposée dans les logs, conteneurs ou variables d'environnement.
  - Test d’accès sécurisé aux App Configuration et Key Vault.

➔ Objectif :
- Garantir que l’infrastructure répond aux besoins de haute disponibilité, sécurité, et performances attendues.


// ===========================================
// COMPOSANTS À DÉPLOYER
// ===========================================

➔ Liste des composants déployés avec leur mode d’autoscaling :

1. API :
   - Autoscalable en fonction de la charge CPU du nœud (HPA).

2. MVC :
   - Autoscalable en fonction de la charge CPU du nœud (HPA).

3. Worker_Content :
   - Autoscalable en fonction du nombre de messages présents dans Service Bus (KEDA).

4. Worker_Image :
   - Autoscalable en fonction du nombre de messages présents dans Service Bus (KEDA).

5. Worker_DB :
   - Autoscalable en fonction du nombre d'événements présents dans Event Hub (KEDA).


// ===========================================
// REMARQUES FINALES
// ===========================================

➔ L'ensemble de la solution respecte les bonnes pratiques suivantes :
- Zéro hardcoding dans les fichiers de configuration ou dans le code source.
- Utilisation des services Azure sécurisés pour toute la gestion de la configuration.
- Pipelines DevOps entièrement automatisés pour le déploiement et la mise à jour.
- Architecture résiliente, scalable, sécurisée, et supervisée.

