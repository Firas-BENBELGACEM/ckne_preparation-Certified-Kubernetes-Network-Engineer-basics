# 5.3 Auditing Traffic with Logs

## Résumé
Confirmé par le retour de beta jkroepke : "Gateway access logs are worth understanding. Be able to distinguish normal, rejected, and suspicious traffic and extract the relevant information quickly."

**Sources de logs pertinentes en Kubernetes networking :**

### A. Logs d'accès Gateway/Envoy (le point clé cité par le retour beta)
Format typique d'un log d'accès Envoy :
```
[2026-09-17T10:15:32.123Z] "GET /api/users HTTP/1.1" 200 - via_upstream 0 1234 15 14 "-" "curl/8.0" "req-id-abc123" "app.example.com" "10.244.1.5:8080"
```
Éléments à savoir lire rapidement :
- Code de statut HTTP (200 = normal, 403 = rejeté par AuthorizationPolicy, 503 = backend indisponible)
- Durée totale de la requête
- IP/port du backend réellement contacté (`upstream_host`)
- `via_upstream` vs autres flags (indique si la requête a atteint le backend ou a été rejetée avant)

```bash
# Récupérer les logs d'accès Envoy d'un Gateway Cilium/Istio
kubectl logs -n <ns> deploy/<gateway-name> -c istio-proxy
kubectl logs -n kube-system -l k8s-app=cilium-envoy
```

### B. Logs Hubble (flux réseau L3/L4/L7 avec verdict)
```bash
hubble observe --namespace default --verdict DROPPED
hubble observe --from-pod default/client --to-pod default/server
hubble observe --protocol http --http-status 403
```
Chaque ligne Hubble montre : source, destination, protocole, port, **verdict** (FORWARDED/DROPPED), et raison en cas de drop.

### C. Distinguer trafic normal / rejeté / suspect
| Type | Signal dans les logs |
|---|---|
| Normal | 2xx, verdict FORWARDED |
| Rejeté (attendu) | 403 (AuthorizationPolicy), verdict DROPPED avec raison "Policy denied" |
| Suspect | Pics de 401/403 depuis une même source, tentatives sur des paths inhabituels (`/admin`, `/.env`), volumétrie anormale depuis une IP |

## Sources officielles
- Hubble CLI : https://docs.cilium.io/en/stable/observability/hubble/
- Envoy Access Logging : https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/access_log
