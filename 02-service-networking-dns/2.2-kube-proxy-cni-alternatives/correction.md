# Correction 2.2

```bash
# 1. État initial
cilium status | grep -i kubeproxy

# 2. Activer le kube-proxy replacement
cilium upgrade --set kubeProxyReplacement=true
cilium status --wait

# 3. Neutraliser kube-proxy classique (si présent en tant que DaemonSet natif)
kubectl -n kube-system get ds kube-proxy
kubectl -n kube-system delete ds kube-proxy
# Nettoyer les anciennes règles iptables laissées par kube-proxy si besoin :
kubectl exec -n kube-system ds/cilium -- iptables-restore <<< "$(iptables-save | grep -v KUBE)"

# 4. Test fonctionnel
kubectl create deployment web --image=nginx --replicas=2
kubectl expose deployment web --port=80
kubectl run client --image=nicolaka/netshoot -- sleep 3600
kubectl exec client -- curl -s http://web

# 5. Observation des tables eBPF de load balancing
kubectl exec -n kube-system ds/cilium -- cilium bpf lb list
```

**Point clé pour l'examen** : sache expliquer la DIFFÉRENCE fonctionnelle (pas seulement dire "eBPF c'est plus rapide") — iptables traverse une liste séquentielle de règles, eBPF utilise une table de hachage à accès direct. C'est la nuance qui distingue une réponse "mémorisée" d'une réponse qui montre une vraie compréhension, ce que le retour de beta valorise explicitement.
