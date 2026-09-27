# Correction 1.4

## Scénario A
```bash
# Diagnostic
kubectl exec client -- nslookup web        # échoue ou timeout
kubectl get pods -n kube-system -l k8s-app=kube-dns   # vérifier CrashLoopBackOff éventuel
kubectl logs -n kube-system -l k8s-app=kube-dns        # erreur de parsing Corefile visible ici

# Réparation : restaurer un Corefile valide
kubectl apply -f - <<EOF2
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
EOF2
kubectl rollout restart deployment coredns -n kube-system

# Validation
kubectl exec client -- nslookup web
```

## Scénario B
```bash
# Diagnostic
kubectl exec client -- curl -m 2 http://app.restricted   # timeout attendu

# La politique deny-all bloque tout ingress. On AJOUTE une règle d'exception
# plutôt que de supprimer la policy existante (bonne pratique zero-trust) :
kubectl apply -n restricted -f - <<EOF2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-default
spec:
  podSelector:
    matchLabels:
      app: app
  policyTypes: ["Ingress"]
  ingress:
    - from:
        - namespaceSelector: {}   # ou plus précis: matchLabels kubernetes.io/metadata.name: default
      ports:
        - protocol: TCP
          port: 80
EOF2

# Validation
kubectl exec client -- curl -m 2 http://app.restricted
```

**Point clé pour l'examen** : dans les deux scénarios, la démarche est identique — reproduire le symptôme, consulter les objets Kubernetes pertinents (`get`/`describe`/`logs`), corriger de façon **ciblée** (ne jamais tout détruire pour repartir de zéro, l'examen valorise probablement les corrections minimales et propres).
