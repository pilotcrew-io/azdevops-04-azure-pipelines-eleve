# Évaluation — 20 points

| Critère | Points | Preuves et attribution |
|---|---:|---|
| Application et tests | 3 | 1 Java17/JAR autonome, 1 santé/404/405 testées, 1 configuration port plateforme et version |
| Vraie CI Azure Pipelines | 4 | 1 fork connecté, 1 Maven task et JUnit visibles, 1 succès, 1 échec intentionnel corrigé |
| Artefact livrable | 3 | 1 ZIP racine app.jar, 1 publication/download même run, 1 SHA vérifié |
| WIF et périmètre | 3 | 1 fédération sans secret, 1 RBAC RG seul, 1 autorisation pipeline + checks connexion |
| CD et approbation | 4 | 1 paramètre/condition manual-main, 1 environment deployment, 1 attente/approuver réelle, 1 santé HTTPS/version |
| Refus, restauration, nettoyage | 3 | 1 matrice skipped/refus, 1 revert contrôlé, 1 suppression et objets hors RG traités |
| **Total** | **20** | |

Rendre le commit évalué, liens de runs (accès privé au formateur), onglet Tests, archive/SHA, captures
masquées de WIF/IAM et checks sur connexion/environnement, états skipped/refusé, santé/version et état
final groupe. Expliquer en deux phrases pourquoi une condition YAML ne protège pas seule une identité.

La compilation locale ne valide que le premier critère et une partie du paquet ; les points Azure
Pipelines et Azure nécessitent des runs/services réels. Indiquer « non vérifié » lorsqu'un accès manque.
Les écrans du corrigé ou sorties attendues n'ont aucune valeur comme preuves personnelles.
Un secret commité exige révocation et traitement de l'incident avant diffusion publique du travail.
