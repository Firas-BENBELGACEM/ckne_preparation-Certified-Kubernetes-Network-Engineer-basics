# 3.2 Implementing Routing to Expose Networks

## Résumé
Ce sujet couvre comment exposer des Services Kubernetes vers l'extérieur du cluster via du **routage réel** (pas juste NodePort) — typiquement BGP ou annonces L2/ARP, déjà vues dans l'eBook Isovalent fourni.

### Option A — BGP (méthode recommandée en datacenter)
Cilium peut établir une session de peering BGP et annoncer les IP LoadBalancer (ou les PodCIDR) au reste du réseau, en s'appuyant sur un device Top-of-Rack.

```yaml
apiVersion: "cilium.io/v2alpha1"
kind: CiliumBGPPeeringPolicy
metadata:
  name: tor
spec:
  nodeSelector:
    matchLabels:
      kubernetes.io/hostname: worker-1
  virtualRouters:
    - localASN: 65001
      exportPodCIDR: true
      neighbors:
        - peerAddress: "172.0.0.1/32"
          peerASN: 65000
```

Vérification :
```bash
cilium bgp peers
# Colonnes: Session State (established), Received/Advertised routes
```

### Option B — L2 Announcements / ARP (petits environnements, home lab)
Cilium répond aux requêtes ARP pour les IP de type LoadBalancer, sans nécessiter de device BGP-capable.

```yaml
apiVersion: "cilium.io/v2alpha1"
kind: CiliumL2AnnouncementPolicy
metadata:
  name: l2-policy
spec:
  serviceSelector:
    matchLabels: {}
  interfaces:
    - ^eth[0-9]+
  externalIPs: true
  loadBalancerIPs: true
```

**Table de décision :**
| Contexte | Méthode |
|---|---|
| Datacenter avec routeurs BGP-capables | BGP (scalable, robuste) |
| Home lab / petit environnement, pas de device BGP | L2 Announcements (ARP) |

## Sources officielles
- Cilium BGP Control Plane : https://docs.cilium.io/en/stable/network/bgp-control-plane/
- Cilium L2 Announcements : https://docs.cilium.io/en/stable/network/l2-announcements/
