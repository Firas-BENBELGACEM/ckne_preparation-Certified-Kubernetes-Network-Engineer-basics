# 2.6 Managing Traffic with the Gateway API (Gateway, HTTPRoutes)

## Résumé
⚠️ **Sujet critique** : cité explicitement dans les DEUX retours de beta disponibles (jkroepke ET le second candidat LinkedIn, qui mentionne spécifiquement "Gateway API avec Envoy"). C'est probablement l'un des sujets les plus lourdement testés du domaine 2 (25% du blueprint).

Le Gateway API remplace progressivement l'ancienne **Ingress API**, qui souffrait d'un manque de portabilité (chaque implémentation d'Ingress Controller ajoutait ses propres annotations non standard — migrer d'un contrôleur à l'autre revenait à "copier une config Juniper sur un routeur Cisco").

**3 objets clés :**

1. **GatewayClass** — définit quel contrôleur gère les Gateways (ex: `cilium`, `istio`, `envoy-gateway`).
2. **Gateway** — instance concrète, définit les listeners (ports, protocoles, certificats TLS).
3. **HTTPRoute** — règles de routage L7 (path, headers, méthode) reliant un Gateway à des Services backend.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: cilium
  listeners:
    - protocol: HTTP
      port: 80
      name: web-gw
      allowedRoutes:
        namespaces:
          from: Same
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-app-1
spec:
  parentRefs:
    - name: my-gateway
      namespace: default
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /details
      backendRefs:
        - name: details
          port: 9080
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: productpage
          port: 9080
```

**Capacités que Gateway API apporte au-delà d'Ingress :** traffic splitting (canary), redirections URL, réécriture de path, mirroring, manipulation de requêtes/réponses HTTP, TLS termination et passthrough — auparavant possibles seulement via annotations propriétaires non-portables.

**Envoy comme dataplane** — plusieurs implémentations de Gateway API (dont Cilium et Istio) utilisent Envoy comme proxy L7 sous-jacent. Comprendre que le `Gateway` est une couche d'abstraction déclarative AU-DESSUS d'Envoy est important pour le troubleshooting (les logs d'accès viennent d'Envoy).

## Sources officielles
- Gateway API : https://gateway-api.sigs.k8s.io/
- Cilium Gateway API : https://docs.cilium.io/en/stable/network/servicemesh/gateway-api/gateway-api/
- Istio Gateway API support : https://istio.io/latest/docs/tasks/traffic-management/ingress/gateway-api/
