# Correction 2.1

```bash
# 1. Service ClusterIP correct
kubectl expose deployment echoserver --port=80 --target-port=8080 --name=echoserver

# 2. Reproduction et diagnostic de la panne
kubectl patch svc echoserver -p '{"spec":{"ports":[{"port":80,"targetPort":8081}]}}'

# Diagnostic SANS events :
kubectl get svc echoserver -o yaml | grep -A3 ports          # targetPort=8081 visible
kubectl get endpointslices -l kubernetes.io/service-name=echoserver -o yaml
# -> les endpoints pointent vers les IP des pods MAIS le port listé est 8081, qui n'existe pas côté conteneur

# Correction
kubectl patch svc echoserver -p '{"spec":{"ports":[{"port":80,"targetPort":8080}]}}'

# 3. NodePort
kubectl expose deployment echoserver --port=80 --target-port=8080 --type=NodePort --name=echoserver-nodeport
kubectl get svc echoserver-nodeport   # noter le port dans la colonne PORT(S), ex 80:31234/TCP

# 4. Test depuis un node (kind = nodes en conteneurs Docker)
NODE_IP=$(docker inspect -f '{{.NetworkSettings.Networks.kind.IPAddress}}' $(kind get nodes | head -1))
curl -s "http://${NODE_IP}:31234" | head -5

# 5. Service sans selector + EndpointSlice manuel
kubectl apply -f - <<EOF2
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
    - port: 443
      targetPort: 443
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-api-1
  labels:
    kubernetes.io/service-name: external-api
addressType: IPv4
ports:
  - port: 443
endpoints:
  - addresses:
      - 192.0.2.10
EOF2

kubectl get endpointslices -l kubernetes.io/service-name=external-api
```

**Points clés pour l'examen :**
- Kubernetes ne crée PAS automatiquement d'EndpointSlice pour un Service sans selector — c'est manuel, avec le label `kubernetes.io/service-name` obligatoire sur l'EndpointSlice.
- Le CNI assigne les IP aux pods, mais c'est le **control plane Kubernetes** qui assigne les IP virtuelles (ClusterIP) aux Services.
