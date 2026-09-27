# Correction 3.2

```bash
# 1. CiliumBGPPeeringPolicy
kubectl apply -f - <<EOF2
apiVersion: "cilium.io/v2alpha1"
kind: CiliumBGPPeeringPolicy
metadata:
  name: bgp-peering
spec:
  nodeSelector:
    matchLabels: {}
  virtualRouters:
    - localASN: 65010
      exportPodCIDR: true
      neighbors:
        - peerAddress: "10.0.0.1/32"
          peerASN: 65000
EOF2

# 2. Vérification
cilium bgp peers
# Attendu : Session State = active/connect (le voisin fictif n'existera pas réellement dans un lab,
# c'est normal — l'objectif est de savoir valider la CONFIGURATION, pas nécessairement obtenir "established")

# 3. LB-IPAM pool
kubectl apply -f - <<EOF2
apiVersion: "cilium.io/v2alpha1"
kind: CiliumLoadBalancerIPPool
metadata:
  name: lb-pool
spec:
  blocks:
    - cidr: "192.168.200.0/24"
EOF2

kubectl create deployment web --image=nginx
kubectl expose deployment web --port=80 --type=LoadBalancer
kubectl get svc web   # doit recevoir une EXTERNAL-IP depuis 192.168.200.0/24

# 4. L2 Announcements
kubectl apply -f - <<EOF2
apiVersion: "cilium.io/v2alpha1"
kind: CiliumL2AnnouncementPolicy
metadata:
  name: l2-default-ns
spec:
  serviceSelector:
    matchLabels: {}
  nodeSelector: {}
  interfaces:
    - ^eth0$
  externalIPs: true
  loadBalancerIPs: true
EOF2

# 5. Validation (nécessite d'être dans le même segment L2, difficile à simuler en kind pur —
# dans un vrai lab, on capturerait avec tcpdump sur l'hôte) :
kubectl exec -n kube-system ds/cilium -- cilium monitor --type l2-announcement 2>/dev/null || \
  kubectl exec -n kube-system ds/cilium -- tcpdump -i eth0 -n arp -c 10
```

**Point clé pour l'examen** : sache identifier laquelle des deux méthodes (BGP vs L2) répondre selon le contexte donné dans l'énoncé — un énoncé mentionnant "pas de device BGP-capable" ou "petit environnement" pointe vers L2 Announcements ; un énoncé mentionnant "datacenter", "scalabilité", "Top-of-Rack" pointe vers BGP.
