# Dépannage — distinguer build, accès et démarrage

| Symptôme | Lecture ciblée | Correction attendue |
|---|---|---|
| pipeline introuvable dans GitHub | installation App et dépôt sélectionné | autoriser uniquement le fork voulu, vérifier la branche et le chemin YAML |
| agent reste queued | pool et capacité parallel jobs | utiliser la capacité préparée avant cours, pas modifier le Java |
| Maven compile avec une autre version | log Maven/JAVA_HOME/java -version | sélectionner JDKVersion1.17 et release17 |
| aucun test JUnit visible | `target/surefire-reports`, nom des tests | tests `*Test`, publication JUnit activée, ne pas ignorer test failures |
| archive contient target/app.jar | liste `unzip -l` | archiver le répertoire préparé sans inclure sa racine |
| CD skipped inattendu | reason, SourceBranch, paramètre et CI | choisir Run pipeline main deploy=true après succès, jamais enlever les garde-fous |
| ressource non autorisée | message Azure DevOps sur connexion/env | administrateur autorise cette seule pipeline après les checks |
| WIF ne peut être créée | droits app Entra/RBAC | faire préparer l'identité par administrateur, aucun secret d'abonnement de secours |
| fédération AADSTS échoue | issuer/subject/audience exacts, tenant | recopier wizard courant, vérifier credential et propagation sans imprimer le jeton |
| AuthorizationFailed Azure | IAM identité sur RG exact | affectation limitée ou ressource hors RG ; corriger périmètre, pas élargir à abonnement |
| contrôle branche refuse | nom fully qualified et main protégée | `refs/heads/main`, branch rules, statut unknown => examiner les paramètres |
| attente approbation | checks connexion ET environnement | approbateur distinct examine chaque demande, ne pas enlever le contrôle |
| aucun package trouvé | nom artefact et workspace dans job CD | télécharger app du même run au chemin prévu |
| 502 après succès task | logs démarrage, contenu ZIP, port | manifeste, startup app.jar, SERVER_PORT, JVM17 ; attendre avec borne |
| 404 /health | logs et route réelle | chemin exact et code déployé, ne pas considérer déploiement API=service sain |
| nom Web App pris | disponibilité du nom avant création | choisir autre suffixe court approuvé, garder RG dédié |

## Lire l'état sans dévoiler de secrets

```bash
az webapp show --resource-group "$RG" --name "$APP_NAME"   --query '{state:state,host:defaultHostName}' -o table
az webapp config show --resource-group "$RG" --name "$APP_NAME"   --query '{runtime:linuxFxVersion,startup:appCommandLine}' -o table
az webapp log tail --resource-group "$RG" --name "$APP_NAME"
```

Pour les logs de démarrage, activer si nécessaire Application logging filesystem dans la Web App
ou via CLI documentée ; les logs peuvent contenir des informations privées. Ne jamais imprimer un
profil de publication, les variables secrètes ou le jeton Azure. Si le test de santé épuise son attente,
le run doit échouer et le compte rendu doit indiquer que la mise en service n'est pas démontrée.
