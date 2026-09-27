# Mock Exam 2 — Focus sujets à risque (LLM traffic, Multi-cluster, Multi-interface)

Ces 3 sujets sont les moins documentés/les plus récents du blueprint — c'est ici que la méthode "comprendre + savoir chercher la doc" compte le plus, plutôt que la mémorisation.

**Temps cible total : 60 minutes**

## Tâche 1 (~20 min)
Sans relire le cours au préalable, va directement sur https://gateway-api-inference-extension.sigs.k8s.io/ et :
1. Identifie la structure actuelle de la CRD `InferencePool`.
2. Écris un `HTTPRoute` routant `/v1/chat/completions` vers un `InferencePool` nommé `chat-pool`.
3. Explique en 2 phrases pourquoi un Service ClusterIP classique serait insuffisant ici.

## Tâche 2 (~20 min)
1. Écris une `CiliumNetworkPolicy` autorisant les pods `app=api` du `cluster-eu` à contacter les pods `app=db` du `cluster-us` sur le port 5432.
2. Liste les 3 prérequis techniques à Cluster Mesh, sans les relire dans le cours (test de mémoire active).
3. Explique pourquoi cette policy n'autorise pas le sens inverse.

## Tâche 3 (~20 min)
1. Installe Multus sur un cluster Cilium existant.
2. Crée une `NetworkAttachmentDefinition` de type `macvlan` sur l'interface `eth0`.
3. Attache ce réseau secondaire à un pod et vérifie ses 2 interfaces.
4. Explique le rôle de "meta-plugin" de Multus en une phrase.

## Auto-évaluation
Pour chaque tâche, note : as-tu dû consulter la doc officielle (normal et attendu), ou as-tu bloqué complètement sans savoir où chercher (signal qu'il faut retravailler le cours correspondant) ?
