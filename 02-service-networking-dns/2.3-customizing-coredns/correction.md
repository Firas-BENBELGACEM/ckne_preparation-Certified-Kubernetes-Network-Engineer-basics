# Correction 2.3

```bash
# 1 + 3. Modification du Corefile (stub domain + cache TTL)
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
        cache 60
        loop
        reload
        loadbalance
    }
    corp.internal:53 {
        errors
        cache 30
        forward . 10.20.0.5
    }
EOF2

# 2. Redémarrage et validation de la config chargée
kubectl rollout restart deployment coredns -n kube-system
kubectl rollout status deployment coredns -n kube-system
kubectl exec -n kube-system deploy/coredns -- cat /etc/coredns/Corefile

# 4. Pod avec DNS custom
kubectl apply -f - <<EOF2
apiVersion: v1
kind: Pod
metadata:
  name: custom-dns-pod
spec:
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 8.8.8.8
    options:
      - name: ndots
        value: "2"
  containers:
    - name: app
      image: nicolaka/netshoot
      command: ["sleep", "3600"]
EOF2

kubectl exec custom-dns-pod -- cat /etc/resolv.conf   # doit montrer nameserver 8.8.8.8 uniquement
kubectl exec custom-dns-pod -- nslookup google.com
```

**Point clé pour l'examen** : une modification du Corefile SANS `kubectl rollout restart` (ou équivalent reload) ne sera PAS appliquée — CoreDNS ne recharge pas automatiquement la ConfigMap montée, sauf si le plugin `reload` est actif dans le Corefile (il l'est par défaut et détecte les changements après quelques secondes, mais un restart explicite est plus fiable et rapide en contexte d'examen chronométré).
