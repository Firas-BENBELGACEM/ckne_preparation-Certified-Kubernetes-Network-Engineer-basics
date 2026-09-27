# Exercice 1.5 — Pods multi-interfaces avec Multus

**Temps cible : 20 minutes**

## Contexte
Une application legacy doit recevoir du trafic sur un second réseau isolé (simulant un réseau "management" séparé du trafic applicatif normal) en plus de son interface Cilium standard.

## Tâches
1. Installe Multus CNI sur ton cluster kind (déjà équipé de Cilium comme CNI primaire).
2. Crée une `NetworkAttachmentDefinition` de type `bridge` nommée `mgmt-net` avec un CIDR `192.168.100.0/24`.
3. Déploie un pod avec l'annotation appropriée pour attacher ce réseau secondaire.
4. Vérifie avec `kubectl exec <pod> -- ip addr show` que le pod dispose bien de 2 interfaces (`eth0` géré par Cilium et `net1` géré par Multus).
5. Explique en une phrase pourquoi Multus doit être positionné comme "meta-plugin" plutôt que comme un CNI parmi d'autres.
