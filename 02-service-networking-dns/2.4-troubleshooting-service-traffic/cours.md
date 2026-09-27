# 2.4 Troubleshooting Service Network Traffic

## Résumé
Méthodologie de diagnostic en couches pour un Service qui ne fonctionne pas, du plus haut niveau au plus bas :

```
1. Le Service existe-t-il et a-t-il une ClusterIP ?
   kubectl get svc <name>

2. A-t-il des Endpoints/EndpointSlices sains ?
   kubectl get endpointslices -l kubernetes.io/service-name=<name>
   → Si VIDE : le selector du Service ne matche aucun pod, ou les pods ne sont pas Ready

3. Le port/targetPort sont-ils cohérents avec le port d'écoute réel du conteneur ?
   kubectl get svc <name> -o yaml | grep -A3 ports
   kubectl exec <pod> -- ss -tlnp   (ou netstat -tlnp)

4. Une NetworkPolicy bloque-t-elle le trafic vers les pods backend ?
   kubectl get networkpolicy,ciliumnetworkpolicy -A

5. kube-proxy / Cilium a-t-il bien programmé les règles de load balancing ?
   cilium service list
   cilium bpf lb list

6. Le trafic atteint-il physiquement le node/pod (capture réseau) ?
   cilium monitor --type drop
```

**Symptômes fréquents et diagnostics associés :**
| Symptôme | Cause probable |
|---|---|
| Connection refused immédiat | targetPort ne correspond à rien côté conteneur |
| Timeout (pas de réponse) | NetworkPolicy bloque, ou pods pas Ready, ou pas d'endpoints |
| Résout mais 0 endpoint | selector du Service ne matche pas les labels des pods |
| Fonctionne parfois, pas toujours | Un sous-ensemble de pods est unhealthy (readinessProbe) — voir 2.5 |

## Sources officielles
- Debug Services : https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/
- Cilium service troubleshooting : https://docs.cilium.io/en/stable/operations/troubleshooting/#service-load-balancing-debugging
