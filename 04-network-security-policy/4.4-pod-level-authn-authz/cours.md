# 4.4 Implementing Pod-level Authentication and Authorization

## Résumé
Confirmé explicitement par le retour de beta jkroepke comme point à maîtriser : "focus on Istio workload security. PeerAuthentication defines how workloads authenticate to each other, while AuthorizationPolicy controls which requests are allowed. Understanding authentication versus authorization matters, not memorizing every field."

**Distinction fondamentale à ne jamais confondre à l'examen :**
- **Authentication (PeerAuthentication)** = QUI es-tu ? (vérification d'identité entre workloads, via mTLS)
- **Authorization (AuthorizationPolicy)** = QUE peux-tu faire ? (une fois identifié, quelles actions/requêtes sont permises)

### PeerAuthentication (Istio) — mTLS entre workloads
```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT   # ou PERMISSIVE (accepte mTLS et plain text), ou DISABLE
```
Modes possibles : `STRICT` (mTLS obligatoire), `PERMISSIVE` (accepte les deux, utile en migration progressive), `DISABLE`.

### AuthorizationPolicy (Istio) — contrôle d'accès
```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/default/sa/frontend"]
      to:
        - operation:
            methods: ["GET"]
            paths: ["/api/*"]
```
Le champ `principals` référence l'**identité SPIFFE** du service account source — c'est le lien direct avec l'authentification mTLS (l'identité vérifiée par PeerAuthentication devient utilisable dans les règles d'AuthorizationPolicy).

### Alternative : SPIFFE/SPIRE (identité de workload, hors service mesh)
SPIFFE (Secure Production Identity Framework For Everyone) fournit des identités cryptographiques (SVID - SPIFFE Verifiable Identity Document) à chaque workload, indépendamment d'un service mesh complet — utilisé notamment par Cilium pour de l'authentification mutuelle fine.

**Diagnostic d'une panne d'autorisation Istio :**
```bash
kubectl get peerauthentication -A
kubectl get authorizationpolicy -A
istioctl analyze                     # détecte les incohérences de config
kubectl logs <pod> -c istio-proxy | grep -i "RBAC"   # logs Envoy montrent les DENY explicites
```

## Sources officielles
- Istio PeerAuthentication : https://istio.io/latest/docs/reference/config/security/peer_authentication/
- Istio AuthorizationPolicy : https://istio.io/latest/docs/reference/config/security/authorization-policy/
- SPIFFE : https://spiffe.io/docs/latest/spiffe-about/overview/
