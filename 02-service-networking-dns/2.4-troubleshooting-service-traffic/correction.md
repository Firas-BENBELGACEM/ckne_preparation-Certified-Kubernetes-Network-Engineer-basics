# Correction 2.4

```bash
# 1. Diagnostic : pas d'endpoints
kubectl get endpointslices -l kubernetes.io/service-name=checkout
# -> vide, car selector du Service (app=checkout-svc) ne matche pas le label réel des pods (app=checkout)

kubectl get pods --show-labels -l app=checkout

# 2. Correction du selector
kubectl patch svc checkout -p '{"spec":{"selector":{"app":"checkout"}}}'
kubectl get endpointslices -l kubernetes.io/service-name=checkout   # doit maintenant lister 3 endpoints

# 3. NetworkPolicy deny-all puis correction ciblée
kubectl apply -f - <<EOF2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes: ["Ingress"]
EOF2

kubectl run client --image=nicolaka/netshoot -- sleep 3600
kubectl exec client -- curl -m 2 http://checkout   # timeout attendu

kubectl apply -f - <<EOF2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-checkout
spec:
  podSelector:
    matchLabels:
      app: checkout
  policyTypes: ["Ingress"]
  ingress:
    - from: []
      ports:
        - protocol: TCP
          port: 80
EOF2

# 4. Validation finale
kubectl exec client -- curl -s http://checkout
```

**Point clé pour l'examen** : cet exercice combine volontairement 2 domaines (Service networking + Network Policy) car c'est exactement le type de scénario multi-couches rapporté par les candidats beta — l'examen ne teste jamais un item isolément, il enchaîne les couches. Entraîne-toi à ne jamais t'arrêter au premier fix trouvé sans revalider de bout en bout.
