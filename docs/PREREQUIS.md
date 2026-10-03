# Prérequis — avant les 265 minutes

## Poste et comptes

Git, JDK17, Maven3.9.x, curl, unzip, Python3 et Azure CLI. Vérifier `git --version`, `java -version`,
`mvn -version`, `az version`. Maven doit afficher le même Java17. Les liens d'installation sont dans
SOURCES. Le projet Maven est créé dans cet atelier : aucun JAR ou dépôt d'atelier01 n'est requis.
Un compte GitHub permettant de forker le dépôt élève public est nécessaire.

Un compte **Azure DevOps Services**, une organisation et un projet **privé** sont nécessaires :
les nouveaux projets publics Azure DevOps ne sont plus un parcours de création disponible.
Avoir les droits de créer une pipeline, un environnement et d'autoriser une connexion ; sinon le
formateur les prépare. Confirmer avant le cours l'accès à un agent Microsoft-hosted Ubuntu et un
job parallèle. Les organisations nouvelles peuvent nécessiter une demande ou une offre payante ;
une attente d'agent ne doit pas consommer la séance. Un agent self-hosted dédié est possible seulement
si fourni/administré par l'établissement, sans mélange de builds PR non fiables et de déploiements.

## Azure : accès et budget préalables

Abonnement actif, région et quota App Service validés, budget accepté pour un plan Linux B1 ou autre
SKU autorisé. Le coût dépend de région, offre et durée ; B1 n'est pas présumé gratuit. Pas de slot
requis. Préparer un groupe dédié `rg-azd04-<identifiant>` et un nom Web App globalement unique.
Le participant a les droits nécessaires dans ce groupe ; un administrateur habilité prépare les
identités et affectations RBAC. L'identité de pipeline a **Contributor sur ce seul groupe**, jamais
sur tout l'abonnement. Le rôle donne le déploiement dans ce périmètre, pas le droit d'assigner des rôles.

La connexion Azure Resource Manager emploie **workload identity federation**. Si le participant ne
peut pas créer une app Entra ou une affectation RBAC, le formateur les prépare avant la séance.
Ne pas contourner un refus avec un secret Contributor abonnement ou un profil de publication.
L'automatisation de création de la connexion peut demander un administrateur Owner de l'abonnement ;
ce droit n'est pas attribué à l'élève ni à l'identité de pipeline.

```bash
az login
az account set --subscription 'ID_ABONNEMENT_AUTORISE'
az account show --query '{name:name,id:id,tenant:tenantId}' -o table
az provider show --namespace Microsoft.Web --query registrationState -o tsv
az webapp list-runtimes --os linux -o tsv
```

Faire confirmer un runtime JavaSE17 (CLI généralement `JAVA:17-java17` ; les tâches utilisent `JAVA|17-java17`).
Si absent ou interdit, demander une adaptation au formateur avant le cours. Aucun cloud n'a été créé
par ce support. La compilation locale ne valide pas un déploiement Azure.

## Fork et accès Azure Pipelines

Forker `pilotcrew-io/azdevops-04-azure-pipelines-eleve` quand il est publié. Cloner votre fork ; développer
sur une branche et conserver `main` comme branche de livraison revue. Le corrigé formateur privé ne
doit pas être rendu public. Avant publication, utiliser les archives fournies.
Dans Azure DevOps, installer/autoriser l'application GitHub Azure Pipelines **sur le seul dépôt de
l'atelier** puis créer une pipeline via `Pipelines > New pipeline > GitHub`. Le code reste sur GitHub ;
le moteur CI/CD est Azure DevOps. « Importer » ici signifie connecter le fork, pas recopier une GitHub
Action. Une importation vers Azure Repos est facultative et change la configuration de validation PR
(branch policy) : elle n'est pas le chemin évalué.

Un approbateur distinct (binôme/formateur) et un administrateur de ressources approuvées sont prêts
avant le cours. Activer le contrôle de branche sur la connexion avant de l'autoriser à la pipeline.
Les PR de forks externes restent désactivées pour ce laboratoire ; aucune permission/secrets n'est
accordée à du code PR non fiable. Les PR du fork de travail sont testées en CI uniquement.

## Empreintes sur macOS

Les agents Ubuntu et les commandes Linux utilisent `sha256sum`. Sur macOS, installer
GNU coreutils via le gestionnaire approuvé ou remplacer `sha256sum fichier` par
`shasum -a 256 fichier`, et `sha256sum --check SHA256SUMS` par
`shasum -a 256 -c SHA256SUMS`. Le format de manifeste SHA256 est compatible.
