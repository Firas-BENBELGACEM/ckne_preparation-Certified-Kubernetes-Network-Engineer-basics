# 3.3 Configuring Egress Gateways for Cluster Exit Traffic

## Résumé
Cité explicitement dans la série Medium de préparation CKNE fournie en référence. Problème résolu : les firewalls traditionnels (Palo Alto, CheckPoint, Cisco) n'ont **aucune connaissance des objets Kubernetes** (namespace, labels). Quand un paquet quitte le cluster, sa source IP est masquée (source-NAT) avec l'IP du node — le firewall ne peut donc pas distinguer quel pod/namespace a émis le trafic.

**Egress Gateway = Source-NAT conscient de Kubernetes.** Pour les pods d'un namespace/label donné, on force la sortie du trafic via un node spécifique, NATé avec une IP dédiée — permettant au firewall d'appliquer une règle précise basée sur cette IP.

```yaml
apiVersion: cilium.io/v2
kind: CiliumEgressGatewayPolicy
metadata:
  name: egress-sample
spec:
  selectors:
    - podSelector:
        matchLabels:
          org: empire
          io.kubernetes.pod.namespace: default
  destinationCIDRs:
    - "0.0.0.0/0"
  egressGateway:
    nodeSelector:
      matchLabels:
        node.kubernetes.io/name: a-specific-node
    egressIP: 10.168.60.100
```

**Ce que ça permet concrètement** : tout le trafic sortant des pods `org=empire` dans `default` ressortira avec la source IP `10.168.60.100`, permettant au firewall externe d'appliquer une règle du type `allow src=10.168.60.100 dst=10.24.0.5`.

**Limite connue** : le node egress devient un point de défaillance unique (SPOF). L'édition entreprise de Cilium propose une HA multi-nodes en round-robin, mais ce n'est pas dans l'édition open source — bon à savoir si une question distingue OSS vs Enterprise.

## Sources officielles
- Cilium Egress Gateway : https://docs.cilium.io/en/stable/network/egress-gateway/egress-gateway/
