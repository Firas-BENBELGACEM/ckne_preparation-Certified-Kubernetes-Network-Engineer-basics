# 3.4 Implementing Cross Cluster Service Discovery and Load Balancing

## Résumé
Dans une topologie multi-cluster (plusieurs zones/régions pour la résilience), les clusters n'ont par défaut **aucune connaissance des objets de l'autre** — même si la connectivité IP existe, les Services, pods, namespaces, labels d'un cluster A sont invisibles pour le cluster B.

**Cilium Cluster Mesh** fédère plusieurs instances Cilium sur différents clusters : découverte automatique de Services à travers tous les clusters "meshés" et load balancing du trafic entre eux.

**Cas d'usage n°1 — Haute disponibilité (failover) :**
Des replicas du même Service tournent dans chaque cluster. Le trafic est par défaut load-balancé sur TOUS les endpoints à travers les clusters. Si tous les endpoints d'un cluster tombent, le trafic bascule automatiquement vers les endpoints des autres clusters.

**Cas d'usage n°2 — Network Policy cross-cluster :**
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: ingress-to-backend
spec:
  description: "Allow frontend in cluster-1 to contact backend in cluster2"
  endpointSelector:
    matchLabels:
      name: backend
      io.cilium.k8s.policy.cluster: cluster-2
  ingress:
    - fromEndpoints:
        - matchLabels:
            name: frontend
            io.cilium.k8s.policy.cluster: cluster-1
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
```

Notez le label spécial `io.cilium.k8s.policy.cluster` qui permet de cibler des endpoints d'un cluster spécifique dans une politique cross-cluster.

**Prérequis techniques pour Cluster Mesh** : connectivité IP directe entre les clusters (souvent via VPN/peering cloud), CA (Certificate Authority) partagée ou fédérée entre clusters, et des CIDR de pods qui ne se chevauchent PAS entre clusters.

## Sources officielles
- Cilium Cluster Mesh : https://docs.cilium.io/en/stable/network/clustermesh/clustermesh/
