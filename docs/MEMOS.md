# Mémos — CI, artefacts, CD et identité

## Source GitHub, exécution Azure DevOps

GitHub stocke les commits et PR ; l'application Azure Pipelines permet à Azure DevOps de lire le
fork autorisé et de publier les statuts. Le YAML Azure Pipelines n'est pas un workflow GitHub Actions.
Une branche PR possède du code non livré ; la validation doit exécuter ses tests sans identité cloud.
Un agent Microsoft-hosted est jetable entre jobs. Une organisation peut manquer de capacité parallèle :
une file d'attente n'est pas une erreur de Java. Le projet Azure DevOps de la séance est privé.

## Job, stage et deployment job

Une task est une action fournie par Azure, par exemple Maven@4. Un job réunit des étapes sur un agent.
Un stage organise une partie du processus, avec dépendances et condition. Un deployment job cible un
environnement Azure DevOps et fournit l'historique de déploiement. Cet environnement n'est pas une
Web App Azure et ne crée pas de ressource cloud. Les approbations sont définies sur la ressource dans
l'UI par son administrateur, pas dans le YAML édité par l'auteur du commit.

## Paramètres, variables et expressions

Un paramètre booléen est choisi avant le run et apparaît dans la compilation du YAML (`${{ }}`).
Une variable macro `$(nom)` est substituée pendant les tâches ; ne pas y mettre un secret dans une
commande qui imprime son environnement. La condition `and(...)` teste des valeurs au moment où le
stage peut démarrer. Comparer `Build.SourceBranch` à `refs/heads/main`, pas au nom approximatif d'une
branche. `Build.Reason=Manual` exclut push/PR programmés ; `deploy=false` par défaut prévient une livraison
accidentelle. `succeeded()` garde le blocage en cas de CI en échec.

## Artefact et paquet

Le JAR contient les classes et le manifeste Main-Class. Le ZIP de déploiement contient `app.jar`
à sa racine : inclure `package/` ou `target/` déplacerait le chemin de démarrage. ArchiveFiles@2
avec includeRootFolder=false prépare ce format ; PublishPipelineArtifact conserve les fichiers du
run. Un SHA256 permet de vérifier que le fichier téléchargé n'a pas changé ; il n'authentifie pas à
lui seul l'auteur et n'est pas une signature. Le deployment job télécharge l'artefact du même run et
ne compile pas à nouveau. Les résultats JUnit et l'artefact sont deux objets différents.

## JavaSE sur App Service

Une Web App JavaSE héberge un JAR avec serveur HTTP embarqué. Tomcat hébergerait généralement un WAR.
Configurer Java17 ne transforme pas un JAR quelconque en serveur. Le point d'entrée écoute sur
SERVER_PORT si la plateforme le fournit, puis PORT/local8080. Linux exige un chemin cohérent avec
`/home/site/wwwroot/app.jar` pour le ZIP et la commande de démarrage explicite. Lire `defaultHostName`
au lieu de supposer `<nom>.azurewebsites.net`. Une URL publique réussie complète les tests unitaires.
Une modification du plan et un slot ont des coûts/exigences différents ; aucun slot n'est nécessaire ici.

## Fédération et autorisation

WIF échange un jeton temporaire du contexte de pipeline contre un jeton Entra ; la confiance associe
issuer, subject et audience exacts. L'authentification répond « qui ? », RBAC répond « quelles actions,
où ? ». Un service principal sans secret peut encore être surautorisé : vérifier sa portée RG.
La connexion de service Azure DevOps est une ressource permettant cet échange, pas l'abonnement.
Copier les valeurs du wizard courant évite une hypothèse sur un issuer susceptible d'évoluer.

## Défense contre une PR qui change le YAML

Un auteur PR peut changer la condition YAML. Pour cette raison, le propriétaire de la connexion ajoute
un Branch control main et une Approval dans l'UI de la connexion ; l'environnement possède les mêmes
contrôles. Un accès pipeline limité et une branche main protégée complètent cette séparation. Les PR
externes restent désactivées dans le laboratoire ; ne pas activer leurs secrets ni permissions normales.
Une condition YAML est un contrôle fonctionnel lisible, les checks de ressource sont une politique
administrée séparément. Le propriétaire de ces checks ne doit pas être un contributeur PR non fiable.

## Restauration et coûts

Revert + nouvelle CI/CD rétablit du code connu dans un nouveau build, avec une nouvelle identité de run.
Redéployer exactement un ancien artefact est une autre stratégie, nécessitant sa conservation et un
contrôle du run choisi. Dans les deux cas, vérifier santé/version après livraison. La suppression du
groupe supprime plan et app mais ne supprime ni projet Azure DevOps, ni connexion, ni identité Entra.
