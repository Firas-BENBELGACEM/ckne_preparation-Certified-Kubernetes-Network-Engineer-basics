# Correction 3.3

```bash
# 1. Label du node egress
kubectl label node kind-worker node.kubernetes.io/name=egress-node --overwrite

# 2. CiliumEgressGatewayPolicy
kubectl apply -f - <<EOF2
apiVersion: cilium.io/v2
kind: CiliumEgressGatewayPolicy
metadata:
  name: payments-egress
spec:
  selectors:
    - podSelector:
        matchLabels:
          team: payments
          io.kubernetes.pod.namespace: default
  destinationCIDRs:
    - "0.0.0.0/0"
  egressGateway:
    nodeSelector:
      matchLabels:
        node.kubernetes.io/name: egress-node
    egressIP: 10.168.60.200
EOF2

# 3. Pod de test avec le label
kubectl run payments-test --image=nicolaka/netshoot --labels="team=payments" -- sleep 3600
kubectl exec payments-test -- curl -s ifconfig.me   # (nécessite accès internet sortant du cluster)
# En environnement isolé, valider plutôt via capture sur le node egress :
kubectl exec -n kube-system ds/cilium -- tcpdump -i eth0 -n src 10.168.60.200 -c 5 &
kubectl exec payments-test -- curl -m 2 http://example.com

# 4. Pod SANS le label (contrôle négatif)
kubectl run no-payments-test --image=nicolaka/netshoot -- sleep 3600
kubectl exec no-payments-test -- curl -s ifconfig.me
# -> doit renvoyer l'IP normale du node (masquerading standard), PAS 10.168.60.200
```

**Point clé pour l'examen** : le `podSelector` d'une `CiliumEgressGatewayPolicy` doit inclure explicitement `io.kubernetes.pod.namespace` si tu veux restreindre à un namespace précis — sans cette clause, la policy s'applique à TOUS les pods portant le label dans TOUS les namespaces, ce qui est un piège classique de sur-application non désirée.
