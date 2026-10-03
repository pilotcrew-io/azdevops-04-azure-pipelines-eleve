# Atelier 04 — CI/CD sur Azure DevOps Pipelines

**265 minutes**, pause de 10 minutes incluse. Prérequis réalisés avant la séance.
Vous travaillez dans un fork GitHub et un projet privé Azure DevOps. Le moteur demandé est **Azure
Pipelines**, pas GitHub Actions. Une application minimale est créée ici, sans dépendance à un atelier précédent.

## Contrat produit et livraison

Java17/Maven ; un JAR autonome `app.jar` ; `GET /health` retourne 200 et `ok`, `/version` une version,
une route inconnue retourne404 et les méthodes hors GET retournent405. Le port lit d'abord SERVER_PORT
(App Service), puis PORT (local), sinon8080. Le serveur écoute sur toutes les interfaces. Tests automatiques
pour ces routes, configuration de version/message et démarrage/arrêt propre.

La pipeline YAML nommée `azure-pipelines.yml` déclenche la CI sur push main et PR vers main. Elle
construit et teste avec Java17, publie JUnit et un artefact nommé `app` contenant un ZIP avec `app.jar`
à la racine et une empreinte SHA256. La CD utilise **l'artefact du même run**, sans reconstruire.
Un paramètre booléen de déploiement, faux par défaut, autorise seulement une exécution **manuelle sur
main** réussie. Un deployment job cible un environnement dont l'approbation est définie dans l'UI.
La Web App Linux JavaSE17 est déjà créée ; pas de slot. WIF remplace les secrets.

| Bloc | Durée | Fin cumulée | Production / preuve |
|---|---:|---:|---|
| 1. Application et tests locaux | 35 min | 35 | build Java17, contrat HTTP |
| 2. Fork connecté et CI | 45 min | 80 | pipeline Azure, JUnit, échec volontaire |
| 3. Paquet et artefact | 30 min | 110 | ZIP inspecté, SHA et run |
| 4. Ressources et identité fédérée | 40 min | 150 | Web App, WIF, périmètre RBAC |
| Pause | 10 min | 160 | |
| 5. Environnement protégé et CD | 45 min | 205 | attente, approbation, santé externe |
| 6. Tests négatifs et restauration | 35 min | 240 | PR/branche refusées, retour version |
| 7. Dossier et nettoyage | 25 min | 265 | preuves et suppression vérifiée |

## Mission 1 — Partir d'une application indépendante (35 min)

Créer la structure Maven standard, le serveur HTTP, un pom fixant release17, les versions de plugins
et le manifeste Main-Class. Écrire les tests du contrat. Prévoir APP_MESSAGE et APP_VERSION ; ne pas
insérer de secret dans `/version`. SERVER_PORT prime sur PORT pour respecter le port choisi par la
plateforme. La configuration locale doit rester simple.

```bash
mvn --batch-mode clean verify
java -jar target/app.jar
curl --fail http://127.0.0.1:8080/health
```

Acceptation : tests réussis, service local sans base ni dépendance cloud, JAR exécutable. Faire échouer
un test de route puis corriger pour comprendre le signal. Ne pas commencer par déployer.

## Mission 2 — Connecter GitHub à Azure Pipelines (45 min)

Créer `azure-pipelines.yml`, avec agent Ubuntu Microsoft-hosted, checkout sans persistance des
credentials, triggers main/PR main et Maven@4. Utiliser JDKVersion1.17, JUnit et `verify`. Les tâches
sont fixées à leur version majeure ; expliquer qu'Azure met encore à jour les versions mineures.
Dans `Pipelines > New pipeline > GitHub`, sélectionner votre fork et `Existing Azure Pipelines YAML
file`, branche main et le fichier écrit. Inspecter avant Save/Run. Aucun déploiement ni connexion
Azure n'est nécessaire à la CI de départ. Lire le run, onglet Tests, durée et commit.

Sur une branche `exercise/test-failure`, modifier volontairement une assertion, pousser puis créer
une PR vers main dans **votre propre fork**. Vérifier échec CI et correction par un nouveau commit.
Les PR de forks externes ne sont pas autorisées ; ne pas exposer de secrets à leurs builds.

Acceptation : lien d'un run Azure Pipelines, tests JUnit visibles, échec suivi d'un succès, explication
entre GitHub (source) et Azure DevOps (orchestrateur). Un test uniquement local ne remplit pas ce critère.

## Mission 3 — Publier une livraison identifiable (30 min)

Copier uniquement le JAR exécutable dans un répertoire de préparation. L'archiver en ZIP, sans inclure
le répertoire parent. Publier avec PublishPipelineArtifact@1 ; ajouter SHA256 de `app.zip`. Télécharger
l'artefact depuis le run et vérifier que `app.jar` est à la racine. Le nom `app` et les chemins entre CI
et CD doivent correspondre. Le ZIP contient un JAR, pas un fichier source ni une arborescence target.

Acceptation : archive inspectable, empreinte vérifiée, lien du run et commit correspondant. Question :
que risquerait une reconstruction différente dans la CD ? Définir une durée de rétention appropriée
aux essais pour retrouver la version précédente sans conserver inutilement des ressources.

## Mission 4 — Préparer Web App et WIF (40 min)

Dans le groupe dédié tagué, créer un App Service Plan Linux approuvé et une Web App JavaSE17, nom unique,
HTTPS-only. Relever `defaultHostName` depuis Azure, sans fabriquer l'URL à partir du nom (les noms de
domaine peuvent inclure un suffixe). Le plan et la Web App sont tous deux supprimés avec le groupe.

Dans `Project settings > Service connections`, créer Azure Resource Manager avec fédération
Workload identity. Utiliser la voie automatique seulement avec l'administrateur autorisé ; sélectionner
**le groupe dédié** dans le champ Resource group. Sinon suivre la voie manuelle avec identité Entra
préparée et credential fédéré copié depuis le wizard. Ne coder en dur ni issuer ni subject : copier
les valeurs actuelles exactes et conserver l'audience indiquée. Aucune clé ou client secret.
Désactiver l'accès à toutes les pipelines ; n'autoriser que votre pipeline. Vérifier dans Azure IAM
que Contributor est limité au groupe, sans affectation large héritée sur cette identité.

**Avant autorisation**, le propriétaire de la connexion ajoute dans son UI Approvals and checks un
Branch control `refs/heads/main` (échec si protection inconnue selon le choix du formateur), avec
main protégé, et une Approval par un approbateur distinct. Cette protection reste valable si un YAML
PR est modifié. Une condition YAML seule n'est pas une frontière de sécurité.

Acceptation : nom de connexion, type WIF, périmètre et contrôles prouvés sans secret. Fournir tenant/
client-id seulement au formateur en privé s'il en a besoin pour vérifier l'affectation.

## Mission 5 — Ajouter une CD sélectionnée et approuvée (45 min)

Créer l'environnement `azd04-test` dans Pipelines > Environments avant le premier déploiement.
Dans Approvals and checks, ajouter Approval (binôme/formateur, auto-approbation désactivée, délai60min),
puis Branch control main. Ne pas tenter de mettre l'approbation dans YAML. Restreindre les administrateurs
et autoriser la seule pipeline prévue. Les vérifications de la connexion et de l'environnement peuvent
produire deux demandes d'approbation ; examiner le même run et ses artefacts pour chacune.

Ajouter un stage CD dépendant de CI, conditionnant réussite, raison Manual, branche exacte main et
paramètre deploy=true. Utiliser un deployment job qui télécharge explicitement l'artefact `app` du run
courant, vérifie son SHA, puis AzureWebApp@1 (`webAppLinux`, runtime Java17) vers la Web App préexistante.
Définir le démarrage du JAR à la racine wwwroot, puis contrôler `/health` via HTTPS avec attente bornée.
Les noms Web App/hostname sont des variables non secrètes ; la connexion est un nom de ressource approuvée.

Exécuter `Run pipeline`, main, deploy=true. Observer l'attente de contrôle avant toute écriture Azure.
Le binôme examine commit, Tests et SHA, approuve ; noter réponse santé/version et historique environnement.
Acceptation : stage réellement bloqué puis libéré, JAR du même run, endpoint200. La santé indique la
mise en service, sans prouver un SLA ni une préparation à la production.

## Mission 6 — Prouver les refus et restaurer (35 min)

Observer CD absent (compilation conditionnelle) ou skipped (condition runtime) pour : push main automatique, PR, run manuel main deploy=false, run manuel d'une
branche autre que main deploy=true. Consigner reason/branch/paramètre et état. Aucune connexion Azure
ne doit être appelée dans ces cas. Décrire pourquoi les checks sur la connexion complètent le YAML.
Ne pas faire exécuter un YAML malveillant pour démontrer ce raisonnement.

Publier un changement de message/version après revue ; déployer sur main avec approbation. Simuler
un refus d'approbation et vérifier absence de déploiement. Pour restaurer, créer une **PR de revert**
du changement, merger après CI, puis lancer manuellement le nouveau run main avec deploy=true.
Cette stratégie reconstruit le code précédent ; ce n'est pas un rollback binaire du run ancien.
Extension facultative hors socle : pipeline de livraison indépendante avec sélection d'un ancien
artefact conservé, branch checks et même approbation, ou slot sur un SKU compatible validé.

## Mission 7 — Rendre et nettoyer (25 min)

Rendre les liens des runs positifs/négatifs, captures masquées des checks UI et dossier complété.
Arrêter les runs en attente avant nettoyage. Vérifier nom, owner/workshop et ressources du groupe,
confirmer le nom, puis supprimer ce seul groupe. `az group exists` doit devenir false.
Désactiver/supprimer la connexion de service de l'atelier et son credential fédéré/identité **uniquement
si l'administrateur confirme qu'ils sont dédiés** ; les identités Entra et permissions GitHub ne sont
pas supprimées avec le groupe Azure. Révoquer l'accès GitHub Azure Pipelines au dépôt si inutile.
