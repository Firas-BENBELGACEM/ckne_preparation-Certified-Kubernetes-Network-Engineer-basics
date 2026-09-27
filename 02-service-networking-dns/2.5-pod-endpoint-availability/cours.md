# 2.5 Configuring Pod Endpoint Availability

## Résumé
Un pod n'est ajouté aux **EndpointSlices** d'un Service que s'il est à la fois `Running` ET `Ready`. La disponibilité est pilotée par les **probes Kubernetes** :

| Probe | Rôle | Effet si échec |
|---|---|---|
| `livenessProbe` | Le conteneur est-il vivant ? | Kubernetes redémarre le conteneur |
| `readinessProbe` | Le pod peut-il recevoir du trafic ? | Le pod est retiré des EndpointSlices (mais PAS redémarré) |
| `startupProbe` | Le conteneur a-t-il fini de démarrer ? | Bloque liveness/readiness tant qu'elle échoue |

**C'est le piège classique** : un service "qui répond parfois" est souvent dû à un `readinessProbe` mal calibré qui retire/rajoute des pods du pool de manière intermittente.

```yaml
readinessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 3
```

**Diagnostic :**
```bash
kubectl get pods -o wide   # colonne READY (ex: 0/1 = pas prêt)
kubectl describe pod <name> | grep -A10 Readiness
kubectl get endpointslices -l kubernetes.io/service-name=<svc>   # compare le nombre de pods Ready vs endpoints listés
```

**Autres leviers de disponibilité d'endpoint :**
- `terminationGracePeriodSeconds` + `preStop` hook — pour un drain propre avant suppression du pod, évite de couper des connexions en cours.
- `PodDisruptionBudget` — garantit un nombre minimum de pods disponibles pendant des opérations volontaires (upgrade de node, etc.), pertinent en contexte de troubleshooting de disponibilité de service à l'échelle du cluster.

## Sources officielles
- Configure Liveness/Readiness/Startup Probes : https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- EndpointSlices : https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/
