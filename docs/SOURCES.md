# Sources primaires et vérification

Documentation vérifiée le **3 octobre 2026**. Les libellés d'interface peuvent évoluer ; les noms des
entrées de tâches et le port du runtime doivent être vérifiés pour une nouvelle session.

| Source | URL officielle | Point étayé |
|---|---|---|
| GitHub vers Azure Pipelines | https://learn.microsoft.com/en-us/azure/devops/pipelines/repos/github?view=azure-devops | App GitHub, connexion dépôt, précautions forks, projets publics retirés |
| Connexion Azure Resource Manager | https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=azure-devops | WIF recommandée, portée Resource group, accès pipeline explicite |
| WIF manuelle | https://learn.microsoft.com/en-us/azure/devops/pipelines/release/configure-workload-identity?view=azure-devops | identité, credential federé et droits administrateur |
| Approvals et checks | https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops | contrôles UI, approbations, Branch control pleinement qualifié |
| Conditions | https://learn.microsoft.com/en-us/azure/devops/pipelines/process/conditions?view=azure-devops | succeeded et variables de branche |
| Maven@4 | https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/maven-v4?view=azure-pipelines | JDK1.17, publication JUnit, objectifs |
| ArchiveFiles@2 | https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/archive-files-v2?view=azure-pipelines | archive ZIP sans root folder |
| PublishPipelineArtifact@1 | https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/publish-pipeline-artifact-v1?view=azure-pipelines | artefact du run |
| DownloadPipelineArtifact@2 | https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/download-pipeline-artifact-v2?view=azure-pipelines | artefact current et chemin cible |
| AzureWebApp@1 | https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/azure-web-app-v1?view=azure-pipelines | appType webAppLinux, runtimeStack et startUpCommand |
| JavaSE App Service | https://learn.microsoft.com/en-us/azure/app-service/configure-language-java-deploy-run?pivots=platform-linux | JAR embarqué, runtime, démarrage |
| Variables App Service | https://learn.microsoft.com/en-us/azure/app-service/reference-app-settings | SERVER_PORT et default hostname unique |
| Azure CLI Web App | https://learn.microsoft.com/en-us/cli/azure/webapp?view=azure-cli-latest | runtime CLI, defaultHostName, show/deploy |
| Maven | https://maven.apache.org/guides/getting-started/ | pom et cycle verify |
| Java17 HttpServer | https://docs.oracle.com/en/java/javase/17/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpServer.html | serveur Java minimal |

Durées, barème, noms d'artefact et stratégie de revert sont des décisions pédagogiques. Vérifier les
quotas, tarifs et capacity jobs dans vos offres autorisées ; aucune gratuité n'est promise.
