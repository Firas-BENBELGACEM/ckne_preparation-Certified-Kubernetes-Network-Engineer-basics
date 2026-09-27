# 4.1 Securing Traffic with Network Policies

## Résumé
Équivalent Kubernetes d'une règle de firewall (ACL). Deux niveaux à maîtriser :

### A. NetworkPolicy standard Kubernetes (L3/L4)
Implémentée par le CNI. Filtre par IP/CIDR, namespace, pod selector, port/protocole.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes: ["Ingress", "Egress"]
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except: ["172.17.1.0/24"]
        - namespaceSelector:
            matchLabels: {project: myproject}
        - podSelector:
            matchLabels: {role: frontend}
      ports:
        - {protocol: TCP, port: 6379}
  egress:
    - to:
        - ipBlock: {cidr: 10.0.0.0/24}
      ports:
        - {protocol: TCP, port: 5978}
```
Un `podSelector: {}` vide sélectionne TOUS les pods du namespace.

### B. CiliumNetworkPolicy (identity-based + L7)
Les IP de pods étant éphémères et imprévisibles, Cilium base sa sécurité sur des **identités** dérivées des labels, pas des IP. Au lieu de "autoriser IP X → IP Y", on écrit "autoriser label role=frontend → label role=backend" — la policy s'applique automatiquement aux nouveaux replicas.

**Filtrage L7 (HTTP)** — capacité qu'un NetworkPolicy standard n'a PAS :
```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "rule1"
spec:
  endpointSelector:
    matchLabels: {org: empire, class: deathstar}
  ingress:
    - fromEndpoints:
        - matchLabels: {org: empire}
      toPorts:
        - ports: [{port: "80", protocol: TCP}]
          rules:
            http:
              - {method: "POST", path: "/v1/request-landing"}
```

**Modes d'enforcement Cilium** :
- `default` : les endpoints démarrent sans restriction, mais dès qu'une règle matche un endpoint, tout le trafic non explicitement autorisé pour cet endpoint est bloqué.
- `never` : mode audit-only, tout le trafic passe (pour observer ce qu'une policy capturerait avant de l'appliquer).
- `always` : zero-trust dès le départ, tout est bloqué sauf autorisation explicite.

## Sources officielles
- Network Policies : https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Cilium Network Policy : https://docs.cilium.io/en/stable/security/policy/
- Cilium L7 Policy : https://docs.cilium.io/en/stable/security/policy/layer7/
