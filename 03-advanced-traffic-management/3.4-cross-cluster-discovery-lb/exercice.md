# Exercice 3.4 — Cluster Mesh (conceptuel + policy cross-cluster)

**Temps cible : 20 minutes**

⚠️ Un vrai Cluster Mesh nécessite 2 clusters distincts avec connectivité réseau réelle entre eux — difficile à reproduire rapidement en kind local. Cet exercice se concentre sur la partie **la plus testable à l'examen : écrire une CiliumNetworkPolicy cross-cluster correcte**, en simulant la présence d'un second cluster via les labels.

## Tâches
1. Explique (par écrit) les 3 prérequis techniques indispensables avant de pouvoir activer Cilium Cluster Mesh entre 2 clusters.
2. Écris une `CiliumNetworkPolicy` autorisant les pods `role=frontend` du `cluster-1` à contacter les pods `role=backend` du `cluster-2` sur le port 443, en TCP.
3. Modifie cette policy pour qu'elle soit **asymétrique** : le trafic retour (cluster-2 vers cluster-1) ne doit PAS être autorisé par cette même policy (justifie pourquoi en une phrase — indice : lien avec le modèle de sécurité de Cilium vu en 4.1).
4. Liste les commandes `cilium clustermesh status` et `cilium clustermesh connectivity` — explique ce que chacune permettrait de vérifier si tu avais un vrai environnement multi-cluster.
