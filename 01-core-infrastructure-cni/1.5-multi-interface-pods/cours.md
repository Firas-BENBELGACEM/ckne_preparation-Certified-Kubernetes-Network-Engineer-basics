# 1.5 Configuring Multi-interface Pods

## Résumé
Par défaut, un pod Kubernetes n'a **qu'une seule interface réseau** (`eth0`), gérée par le CNI principal. Certains cas d'usage (réseaux Telco/NFV, séparation trafic data-plane / control-plane, connexion à un VLAN spécifique) nécessitent plusieurs interfaces réseau par pod.

**Multus CNI** est la solution standard : c'est un "meta-plugin" CNI qui permet d'attacher plusieurs interfaces réseau à un pod, en déléguant chaque interface à un plugin CNI différent (le CNI principal du cluster + des CNI secondaires comme macvlan, ipvlan, SR-IOV, etc.).

**Fonctionnement :**
1. Multus est installé comme CNI "meta" — il devient le point d'entrée CNI du cluster.
2. On définit des `NetworkAttachmentDefinition` (CRD) décrivant chaque réseau secondaire disponible.
3. On annote le pod avec `k8s.v1.cni.cncf.io/networks` pour lui attacher une ou plusieurs interfaces supplémentaires.

```yaml
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: macvlan-conf
spec:
  config: '{
    "cniVersion": "0.3.1",
    "type": "macvlan",
    "master": "eth1",
    "mode": "bridge",
    "ipam": {
      "type": "host-local",
      "subnet": "192.168.1.0/24"
    }
  }'
---
apiVersion: v1
kind: Pod
metadata:
  name: multi-net-pod
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf
spec:
  containers:
    - name: app
      image: nginx
```

Commande pour vérifier les interfaces d'un pod multi-réseau :
```bash
kubectl exec multi-net-pod -- ip addr show
# eth0 = interface CNI principale (Cilium), net1 = interface Multus secondaire
```

## Sources officielles
- Multus CNI : https://github.com/k8snetworkplumbingwg/multus-cni
- Network Plumbing Working Group (spec NetworkAttachmentDefinition) : https://github.com/k8snetworkplumbingwg/multi-net-spec
