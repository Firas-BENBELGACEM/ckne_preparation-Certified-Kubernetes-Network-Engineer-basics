# Correction 4.4

```bash
# 1. PeerAuthentication STRICT
kubectl apply -f - <<EOF2
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
EOF2

# 2. Validation (le sidecar gère le mTLS de façon transparente)
kubectl exec deploy/frontend -c nginx -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" http://backend

# 3. AuthorizationPolicy restrictive
kubectl apply -f - <<EOF2
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: backend-allow-get-from-frontend
  namespace: default
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/default/sa/default"]   # SA par défaut si non spécifié explicitement
      to:
        - operation:
            methods: ["GET"]
EOF2

# 4. Test POST (doit échouer en 403)
kubectl exec deploy/frontend -c nginx -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" -X POST http://backend

# 5. Pod mallory (SA différent, non autorisé même en GET si SA distinct de celui utilisé dans la policy)
kubectl create serviceaccount mallory-sa
kubectl run mallory --image=nicolaka/netshoot --overrides='{"spec":{"serviceAccountName":"mallory-sa"}}' -- sleep 3600
kubectl exec mallory -- curl -m 2 -s -o /dev/null -w "%{http_code}\n" http://backend
# Attendu : 403, car le principal SPIFFE de mallory-sa ne matche pas la policy
```

**Réponse à la question 6** : une requête bloquée par `PeerAuthentication` échoue AU NIVEAU DE LA CONNEXION TLS elle-même (le handshake mTLS échoue car l'identité du client ne peut pas être vérifiée cryptographiquement — la connexion n'est jamais établie). Une requête bloquée par `AuthorizationPolicy` réussit d'abord son handshake mTLS (l'identité EST vérifiée avec succès), mais la requête applicative est ensuite explicitement rejetée par Envoy avec un code HTTP 403, car l'action demandée (méthode/path) n'est pas dans la liste des règles `ALLOW`. C'est exactement la distinction authentication (qui es-tu) vs authorization (que peux-tu faire) soulignée par le retour de beta.
