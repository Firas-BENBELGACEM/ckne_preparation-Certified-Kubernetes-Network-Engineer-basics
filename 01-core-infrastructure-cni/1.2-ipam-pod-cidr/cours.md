# 1.2 Managing IPAM and Pod CIDR Allocation

## Résumé
En réseau traditionnel, le DHCP alloue les IP. En Kubernetes, il n'y a pas de DHCP pour les pods : c'est le rôle de l'**IPAM (IP Address Management)**, généralement géré par le CNI. Un pod reçoit son IP depuis un sous-réseau appelé **PodCIDR**.

Cilium propose 3 modes IPAM principaux :

| Mode | Qui assigne les CIDR | Multi-CIDR/cluster | Multi-CIDR/node | Allocation dynamique |
|---|---|---|---|---|
| Kubernetes Host Scope | Kubernetes | ❌ | ❌ | ❌ |
| Cluster Scope | Cilium | ✅ | ❌ | ❌ |
| Multi-Pool | Cilium | ✅ | ✅ | ✅ |

- **Kubernetes Host Scope** : un préfixe cluster-wide, découpé en sous-réseau par node. Si le préfixe est épuisé, impossible d'agrandir le pool.
- **Cluster Scope** : Cilium assigne les CIDR par node et permet d'ajouter des préfixes supplémentaires si le pool s'épuise.
- **Multi-Pool** : le plus flexible — permet d'allouer depuis plusieurs pools IPAM différents selon des propriétés du workload (annotations), avec ajout dynamique de PodCIDR à un node.

**Commandes utiles :**
```bash
# Voir le PodCIDR par node
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.podCIDR}{"\n"}{end}'

# Configuration Cilium (ConfigMap)
kubectl get -n kube-system configmap cilium-config -o yaml | grep -i ipam

# Via le Cilium CLI
cilium config view | grep ipam
```

Note importante : les pods de **namespaces différents** sur le **même node** peuvent recevoir des IP du **même PodCIDR** — la segmentation par namespace n'est pas une segmentation réseau (contrairement à ce qu'on pourrait attendre en venant du monde VLAN).

## Sources officielles
- Cilium IPAM : https://docs.cilium.io/en/stable/network/concepts/ipam/
- Kubernetes Networking Model : https://kubernetes.io/docs/concepts/cluster-administration/networking/
