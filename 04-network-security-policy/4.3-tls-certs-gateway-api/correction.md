# Correction 4.3

```bash
# 1. ClusterIssuer self-signed
kubectl apply -f - <<EOF2
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
EOF2

# 2. Certificate
kubectl apply -f - <<EOF2
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-cert
  namespace: default
spec:
  secretName: app-tls
  dnsNames:
    - "app.ckne.local"
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
EOF2

# 3. Vérification
kubectl get certificate app-cert
kubectl wait --for=condition=Ready certificate/app-cert --timeout=60s
kubectl get secret app-tls

# 4. Gateway avec listener HTTPS
kubectl apply -f - <<EOF2
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: tls-gateway
spec:
  gatewayClassName: cilium
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        mode: Terminate
        certificateRefs:
          - name: app-tls
      allowedRoutes:
        namespaces:
          from: Same
EOF2

kubectl get gateway tls-gateway   # PROGRAMMED=True attendu

# 5. Panne volontaire
kubectl patch gateway tls-gateway --type=json -p='[{"op": "replace", "path": "/spec/listeners/0/tls/certificateRefs/0/name", "value": "wrong-secret-name"}]'
kubectl describe gateway tls-gateway
# Attendu dans les conditions : un message d'erreur type "Secret wrong-secret-name not found" ou
# "certificateRefs" invalide, statut PROGRAMMED probablement False ou listener en erreur

# 6. Correction
kubectl patch gateway tls-gateway --type=json -p='[{"op": "replace", "path": "/spec/listeners/0/tls/certificateRefs/0/name", "value": "app-tls"}]'
kubectl get gateway tls-gateway   # doit revenir à PROGRAMMED=True
```

**Point clé pour l'examen** : la chaîne complète à vérifier en cas de panne TLS est toujours dans cet ordre : `Certificate READY` → `Secret existe` → `Gateway certificateRefs pointe vers le bon nom de Secret` → `Gateway PROGRAMMED=True`. Ne pas sauter d'étape — un candidat qui va directement modifier le Gateway sans vérifier que le Certificate est bien `READY` risque de perdre du temps sur un faux problème.
