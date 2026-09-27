# 4.3 Managing TLS Certificates for Gateway API

## Résumé
Confirmé explicitement dans le retour de beta jkroepke comme point à maîtriser : "know how a certificate ends up in a Secret and how a Gateway listener references that Secret."

**cert-manager** est l'outil standard CNCF pour automatiser l'émission/renouvellement de certificats TLS dans Kubernetes. Il fonctionne avec des CRD :

- **Issuer / ClusterIssuer** — définit la source des certificats (Let's Encrypt, CA interne, self-signed...)
- **Certificate** — demande de certificat, dont le résultat est stocké dans un **Secret** Kubernetes de type `kubernetes.io/tls`

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: gateway-cert
  namespace: default
spec:
  secretName: gateway-tls-secret
  dnsNames:
    - "my-app.example.com"
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
```

**Le Gateway référence ce Secret dans son listener HTTPS** :
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: cilium
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        mode: Terminate
        certificateRefs:
          - name: gateway-tls-secret     # <- doit matcher secretName du Certificate
      allowedRoutes:
        namespaces:
          from: Same
```

**Chaîne de vérification en cas de panne TLS :**
```bash
kubectl get certificate                         # READY=True ?
kubectl describe certificate gateway-cert        # events en cas d'échec d'émission
kubectl get secret gateway-tls-secret -o yaml    # le Secret existe-t-il vraiment ?
kubectl get gateway my-gateway -o yaml           # le certificateRefs pointe-t-il au bon nom ?
```

**cert-manager pour Gateway API** — utilise une annotation ou intégration native (`cert-manager.io/cluster-issuer`) directement sur la ressource Gateway pour automatiser la création du Certificate sans le déclarer manuellement.

## Sources officielles
- cert-manager : https://cert-manager.io/docs/
- cert-manager + Gateway API : https://cert-manager.io/docs/usage/gateway/
