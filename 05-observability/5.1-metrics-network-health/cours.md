# 5.1 Analyzing Network Health Using Metrics

## Résumé
Confirmé par le retour de beta jkroepke : "Prometheus and Jaeger UIs were more relevant [than low-level tools]. Provided metrics or traces can be used to identify slow, failing, or otherwise unhealthy services **without memorizing metric names or writing complex PromQL from scratch**." → l'examen semble tester la capacité à NAVIGUER une UI Prometheus déjà déployée plutôt qu'à écrire du PromQL avancé de mémoire.

**Hubble Metrics** (Cilium) — exporte des métriques réseau natives vers Prometheus :
```bash
cilium install --set hubble.metrics.enabled="{dns,drop,tcp,flow,icmp,http}"
```
Métriques clés exposées : `hubble_drop_total` (paquets droppés, avec raison), `hubble_flows_processed_total`, `hubble_http_requests_total`, `hubble_dns_queries_total`.

**Workflow attendu à l'examen (d'après le retour de beta) :**
1. Ouvrir l'UI Prometheus déjà déployée
2. Chercher/filtrer les métriques pertinentes pour le service concerné (pas besoin de connaître le nom exact par cœur — utiliser l'autocomplétion de l'UI)
3. Identifier le pod/service en anomalie
4. Traduire cette observation en action Kubernetes concrète (redémarrage, correction de policy, etc.)

```bash
# Exemples de requêtes PromQL simples à savoir reconnaître (pas nécessairement écrire de zéro)
rate(hubble_drop_total[5m])
sum by (destination) (rate(hubble_http_requests_total{status_code=~"5.."}[5m]))
histogram_quantile(0.95, rate(hubble_http_request_duration_seconds_bucket[5m]))
```

**Hubble UI** — alternative graphique à Prometheus pour la même donnée, sous forme de service map (vue déjà présentée dans l'eBook Isovalent fourni) :
```bash
cilium hubble ui   # ouvre l'interface dans le navigateur
```

## Sources officielles
- Hubble Metrics : https://docs.cilium.io/en/stable/observability/metrics/
- Prometheus querying basics : https://prometheus.io/docs/prometheus/latest/querying/basics/
