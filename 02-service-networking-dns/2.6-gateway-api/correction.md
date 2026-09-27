# Correction 2.6

```bash
# 1. Gateway
kubectl apply -f - <<EOF2
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
EOF2

kubectl get gateway my-gateway   # attendre PROGRAMMED=True

# 2. HTTPRoute
kubectl apply -f - <<EOF2
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-app-1
spec:
  parentRefs:
    - name: my-gateway
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
EOF2

# 3. IP externe
kubectl get svc -l io.cilium.gateway/owning-gateway=my-gateway
HTTP_INGRESS=$(kubectl get svc -l io.cilium.gateway/owning-gateway=my-gateway -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}')

# 4. Tests
curl -s "http://${HTTP_INGRESS}/details"
curl -s "http://${HTTP_INGRESS}/"

# 5. Traffic splitting (canary 90/10)
kubectl create deployment productpage-v2 --image=httpd --port=9080
kubectl expose deployment productpage-v2 --port=9080

kubectl apply -f - <<EOF2
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-app-1
spec:
  parentRefs:
    - name: my-gateway
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
          weight: 90
        - name: productpage-v2
          port: 9080
          weight: 10
EOF2

# 6. Header matching
kubectl apply -f - <<EOF2
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: header-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - headers:
            - type: Exact
              name: magic
              value: foo
          path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: productpage-v2
          port: 9080
EOF2

curl -s -H "magic: foo" "http://${HTTP_INGRESS}/"
```

**Points clés pour l'examen :**
- Toujours vérifier `kubectl get gateway <name>` colonne `PROGRAMMED` avant de troubleshooter un HTTPRoute — si le Gateway lui-même n'est pas programmé, aucun HTTPRoute ne fonctionnera.
- `parentRefs` dans un HTTPRoute doit correspondre EXACTEMENT au nom (et namespace si différent) du Gateway — erreur de frappe fréquente = route silencieusement ignorée.
- Les règles de `matches` dans un HTTPRoute sont évaluées dans l'ordre — une règle catch-all (`/`) doit toujours être en DERNIER sous peine de masquer les règles plus spécifiques.
