# 5.2 Troubleshooting End-to-End Network Performance with Tracing

## Résumé
Confirmé comme outil clé dans le retour de beta jkroepke : Jaeger préinstallé dans l'environnement d'examen, cité comme un outil que le candidat "ne connaissait pas vraiment" avant l'examen — donc à découvrir maintenant plutôt qu'à l'examen.

**Le tracing distribué répond à une question que les métriques seules ne peuvent pas** : "Dans une chaîne d'appels A → B → C → D, où exactement se situe la latence ?" Une métrique agrégée montre "le service est lent", une trace montre "c'est spécifiquement l'appel de B vers C qui prend 800ms".

**Jaeger** est l'outil de tracing distribué le plus commun dans l'écosystème CNCF, généralement alimenté par Istio (chaque sidecar Envoy propage automatiquement les headers de trace) ou par instrumentation applicative (OpenTelemetry).

**Concepts clés :**
- **Trace** = le parcours complet d'une requête à travers tous les services
- **Span** = un segment de cette trace (un appel spécifique, avec son propre timing)
- **Trace ID** = identifiant propagé dans les headers HTTP (`x-b3-traceid` en Zipkin format, ou `traceparent` en W3C format) entre tous les services traversés

**Activer le tracing avec Istio :**
```bash
istioctl install --set profile=demo \
  --set meshConfig.defaultConfig.tracing.sampling=100.0 \
  -y
```

**Workflow de diagnostic attendu :**
1. Ouvrir l'UI Jaeger déjà déployée
2. Rechercher les traces du service concerné, filtrées par une durée anormalement longue
3. Ouvrir une trace précise et observer quel span (quel service/appel) concentre la majorité du temps
4. Traduire cette observation en action (le service C est lent → vérifier ses ressources/ses dépendances, PAS le réseau entre B et C si le span réseau lui-même est rapide)

```bash
# Accès à l'UI Jaeger dans un cluster Istio demo profile
kubectl port-forward -n istio-system svc/tracing 16686:80
```

## Sources officielles
- Jaeger : https://www.jaegertracing.io/docs/latest/
- Istio Distributed Tracing : https://istio.io/latest/docs/tasks/observability/distributed-tracing/
