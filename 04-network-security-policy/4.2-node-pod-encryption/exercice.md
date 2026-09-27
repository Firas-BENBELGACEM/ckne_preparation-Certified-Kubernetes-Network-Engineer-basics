# Exercice 4.2 — Chiffrement de trafic pod-to-pod

**Temps cible : 15 minutes**

## Tâches
1. Vérifie l'état actuel du chiffrement sur ton cluster Cilium (probablement désactivé).
2. Active le chiffrement WireGuard.
3. Vérifie via `cilium status` que le chiffrement est actif, et via `cilium encrypt status` (ou équivalent) le nombre de nodes participant au chiffrement.
4. Déploie deux pods sur des nodes DIFFÉRENTS (force le placement avec `nodeSelector` ou `nodeName`), teste la connectivité, puis capture le trafic entre les deux nodes avec tcpdump pour observer qu'il est bien illisible (chiffré) au niveau paquet.
5. Explique en une phrase pourquoi une capture tcpdump DANS un pod (namespace réseau du pod) montrerait quand même le trafic en clair, alors que la capture au niveau de l'interface physique du node le montre chiffré.
