# Correction 5.1

```bash
# 1. Hubble UI
cilium hubble ui
# Naviguer vers le namespace default, observer le flux client -> web

# 2 + 3. Exploration Prometheus (méthode : autocomplétion, pas mémorisation)
# Dans l'UI Prometheus, taper "hubble_http" et laisser l'autocomplétion suggérer :
#   hubble_http_requests_total
# Requête pour le taux d'erreur 4xx :
sum(rate(hubble_http_requests_total{status_code=~"4.."}[5m]))
  /
sum(rate(hubble_http_requests_total[5m]))

# 4. Panne volontaire + observation des drops
kubectl apply -f - <<EOF2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-web
spec:
  podSelector:
    matchLabels: {app: web}
  policyTypes: ["Ingress"]
EOF2

for i in $(seq 1 10); do kubectl exec client -- curl -m 1 -s -o /dev/null http://web/; done

# Dans Prometheus :
sum by (reason) (rate(hubble_drop_total[5m]))
# Doit montrer une raison type "Policy denied"

# 5. Correction
kubectl delete networkpolicy block-web
for i in $(seq 1 10); do kubectl exec client -- curl -m 1 -s -o /dev/null http://web/; done
# Les drops doivent redescendre à 0 dans la métrique
```

**Point clé pour l'examen** : ne PAS paniquer si tu ne te souviens pas du nom exact d'une métrique — l'autocomplétion de l'UI Prometheus (taper `hubble_` et voir la liste) est ta meilleure alliée, exactement comme le souligne le retour de beta ("documentation matters more than memorization").
