# Correction 4.1

```bash
# 1. NetworkPolicy standard
kubectl apply -n shop -f - <<EOF2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-empire-to-deathstar
spec:
  podSelector:
    matchLabels: {class: deathstar}
  policyTypes: ["Ingress"]
  ingress:
    - from:
        - podSelector:
            matchLabels: {org: empire}
      ports:
        - {protocol: TCP, port: 80}
EOF2

# 2. Validation
kubectl exec -n shop deploy/frontend -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" http://deathstar   # 200
kubectl exec -n shop deploy/mallory -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" http://deathstar    # timeout/erreur

# 3. CiliumNetworkPolicy L7
kubectl delete networkpolicy allow-empire-to-deathstar -n shop
kubectl apply -n shop -f - <<EOF2
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "l7-deathstar-policy"
spec:
  endpointSelector:
    matchLabels: {class: deathstar}
  ingress:
    - fromEndpoints:
        - matchLabels: {org: empire}
      toPorts:
        - ports: [{port: "80", protocol: TCP}]
          rules:
            http:
              - {method: "POST", path: "/v1/request-landing"}
EOF2

# 4. Validation L7
kubectl exec -n shop deploy/frontend -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" -X GET http://deathstar/
# Attendu : bloqué (403 ou timeout selon config)

kubectl exec -n shop deploy/frontend -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" -X POST http://deathstar/v1/request-landing
# Attendu : 200 (ou 404 si nginx par défaut n'a pas ce path, mais la requête DOIT atteindre le pod,
# ce qui prouve que la policy L7 a laissé passer)

# 5. Mode audit-only
cilium config set PolicyEnforcementMode never
kubectl exec -n shop deploy/mallory -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" http://deathstar
# En mode "never", même mallory devrait maintenant atteindre deathstar (tout le trafic passe)

# Ne pas oublier de repasser en mode default après le test :
cilium config set PolicyEnforcementMode default
```

**Point clé pour l'examen** : une `CiliumNetworkPolicy` L7 nécessite que le trafic passe par le **proxy Envoy intégré à Cilium** — cela ajoute une latence et un comportement différent d'une simple policy L3/L4 (filtrage eBPF pur). Si un test de connectivité "hang" plus longtemps que d'habitude avec une policy L7 active, c'est normal — ce n'est pas nécessairement une panne.
