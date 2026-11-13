# Tâche 3 du cours IFT3913 : Github actions et vulnérabilités

## Idées de départ

Au tout début, nous voulions démarrer le travail avec des builds "propres" et qui passaient en vert. Plusieurs workflows échouaient déjà, et Dependabot multipliait les exécutions. Dans notre dépôt privé initial, les simples commit d'initialisation de la tâche ont épuisé le quota mensuel de minutes de Github actions disponibles. Sous la panique, nous avons temporairement passé tous les workflows en lancement manuel (par peur de ne plus pouvoir exécuter d'actions pour le prochain mois, et donc de ne pas pouvoir tester la tâche correctement). Heureusement, nous avons pu passé le dépôt en public et rétabli les déclencheurs pour éviter ce fameux quota.

Nous avons aussi tenté de comprendre les erreurs de build et appliqué une correction sur la configuration MinIO des tests d'intégration. Mais réparer les workflows qui échouaient nous éloignait du but du TP et des critères d'évaluation. On sortait du cadre de la tâche. Nous cherchions principalement à bloquer le build sur une nouvelle vulnérabilité et à distinguer ce blocage des autres cas d'échecs.

## Le contrôle retenu

Nous avons d'abord ajouté les étapes de Semgrep directement dans `.github/workflows/main-jdk17-build.yml`, avant Mavent. Cette première version était fonctionnelle, mais nous la trouvions un peu limitée : elle ne protégeait qu'un seul des builds Java. Ce n'était pas trop évident de la manière à prévilégier ou de la couverture attendue dans les consignes de la tâche. Nous voulions donc étendre le contrôle aux autres builds tout en évitant de devoir modifier plusieurs fichiers à chaque correction de Semgrep (centraliser nos modifications). Ceci, en plus de la mention de réutilisabilité dans la consigne du rickroll est ce qui nous a amenés à extraire les étapes dans un workflow réutisable, `.github/workflows/semgrep.yml`. Ainsi, si une correction de l'installation ou de la commande d'analyse se fait à un seul endroit et s'applique à tous les builds Java qui l'appellent.

Le fichier de semgrep déclare `workflow_call`. Chaque workflow JDK l’appelle dans un job `semgrep`, puis impose `needs: semgrep` à son job `build`. Si l’analyse échoue, le workflow échoue et Maven ne démarre pas. Les cinq workflows JDK utilisent ce contrôle. Chaque workflow conserve sa propre exécution du scan. Nous n'avons pas modifié les workflows de publication Docker.

L'analyse choisie dans le setup Semgrep porte sur les cinq modules mentionnés dans le [cas d'étude](https://github.com/umontreal-diro/IFT3913/blob/2026/README.md/#cas-d%C3%A9tude). Nous avons précisé certaines commandes dans `.github/workflows/semgrep.yml` afin d'écarter les alertes déjà dans la référence, faire échouer l'analyse s'il reste une nouvelle alerte et faire aussi remonter certaines erreurs d'analyse comme des échecs. Le checkout récupère l'historique.

Nous avons également fait le choix de garder une référence fixe pour qu'une "nouvelle" (par rapport au commit choisi comme référence) faille  continue de bloquer les commits suviants tant qu'elle est présente. Les éxécutions utilisent le commit `56daa36d5` qui est notre version du dépot avant l'introduction de nouvelles failles.






Plan : 
Problème de build. Github actions ne passent pas, voudrais démarrer avec au moins les 3 build java en vert pour ensuite mieux tester les failles. Essayer de fix les problèmes de build était innutile, on cherche (selon le'énoncé) seulement à bloquer le processus de build lorsqu'on initie la comparaison semgrep. Ajouter semgrep aux autres workflow de build.   Modifier semgrpe pour réutiliser.  J'ai tester semgrep en retirant les nouvelles vulnérabilités et en vérifiant que mon action de vérifications Semgrep était validé et passait au build. J'ai gaspillé 2000 minutes de github actions pour mon mois sur mon repo privé en essayant les tests initial.