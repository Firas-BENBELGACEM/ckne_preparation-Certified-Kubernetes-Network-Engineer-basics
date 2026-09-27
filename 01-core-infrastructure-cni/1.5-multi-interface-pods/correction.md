# Correction 1.5

```bash
# 1. Installation de Multus (méthode thick plugin officielle)
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-thick.yml
kubectl get pods -n kube-system -l app=multus

# 2. NetworkAttachmentDefinition
kubectl apply -f - <<EOF2
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: mgmt-net
spec:
  config: '{
    "cniVersion": "0.3.1",
    "type": "bridge",
    "bridge": "mgmt-br0",
    "isGateway": true,
    "ipam": {
      "type": "host-local",
      "subnet": "192.168.100.0/24"
    }
  }'
EOF2

# 3. Pod avec interface secondaire
kubectl apply -f - <<EOF2
apiVersion: v1
kind: Pod
metadata:
  name: multi-net-pod
  annotations:
    k8s.v1.cni.cncf.io/networks: mgmt-net
spec:
  containers:
    - name: app
      image: nginx
EOF2

# 4. Vérification
kubectl exec multi-net-pod -- ip addr show
# Attendu :
#   eth0  -> IP dans le PodCIDR Cilium (10.244.x.x)
#   net1  -> IP dans 192.168.100.0/24 (interface Multus)
```

**Réponse à la question 5** : Multus doit être le point d'entrée CNI car Kubernetes/kubelet n'appelle qu'**UN SEUL** binaire CNI par pod au moment de la création. Multus se positionne donc comme ce point d'entrée unique et délègue ensuite, en interne, la création de chaque interface à un ou plusieurs CNI "réels" (le CNI primaire du cluster + les CNI secondaires listés dans les NetworkAttachmentDefinition) — c'est le principe du "meta-plugin".
