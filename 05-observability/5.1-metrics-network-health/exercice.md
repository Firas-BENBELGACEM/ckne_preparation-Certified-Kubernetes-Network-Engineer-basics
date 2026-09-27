# Exercice 5.1 — Analyse de métriques réseau

**Temps cible : 15 minutes**

## Setup
```bash
cilium install --set hubble.metrics.enabled="{dns,drop,tcp,flow,http}" --set hubble.relay.enabled=true --set hubble.ui.enabled=true

kubectl create deployment web --image=nginx --replicas=3
kubectl expose deployment web --port=80
kubectl run client --image=nicolaka/netshoot -- sleep 3600

# Génère du trafic incluant des erreurs
for i in $(seq 1 20); do kubectl exec client -- curl -s -o /dev/null http://web/nonexistent-path; done
for i in $(seq 1 20); do kubectl exec client -- curl -s -o /dev/null http://web/; done
```

## Tâches
1. Ouvre Hubble UI et identifie visuellement le flux de trafic entre `client` et `web`.
2. Sans écrire de PromQL de mémoire, utilise l'autocomplétion Prometheus pour trouver la métrique liée aux codes de statut HTTP retournés par Hubble.
3. Identifie le pourcentage de requêtes en erreur (4xx) sur les 5 dernières minutes.
4. Ajoute une NetworkPolicy qui bloque tout le trafic vers `web`, régénère du trafic, et observe l'apparition de `hubble_drop_total` dans Prometheus.
5. Corrige la policy et confirme que les drops s'arrêtent.
