# Exercice 1.4 — Troubleshooting DNS et pod-to-pod

**Temps cible : 20 minutes (2 scénarios de 10 min)**

## Scénario A — DNS cassé
```bash
kubectl create deployment web --image=nginx
kubectl expose deployment web --port=80
kubectl run client --image=nicolaka/netshoot -- sleep 3600
# Casse volontairement CoreDNS
kubectl patch configmap coredns -n kube-system --type merge -p '{"data":{"Corefile":"broken { }"}}'
kubectl rollout restart deployment coredns -n kube-system
```
**Tâche** : diagnostique pourquoi `kubectl exec client -- nslookup web` échoue, et répare.

## Scénario B — Connectivité pod-to-pod bloquée par erreur
```bash
kubectl create namespace restricted
kubectl create deployment app -n restricted --image=nginx
kubectl expose deployment app -n restricted --port=80
kubectl apply -n restricted -f - <<POLICY
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes: ["Ingress"]
POLICY
```
**Tâche** : un autre pod (`client`, namespace default) doit pouvoir joindre `app.restricted` sur le port 80. Diagnostique et corrige sans supprimer la politique deny-all (ajoute une règle d'exception).
