# Correction 3.4

## 1. Prérequis techniques
1. **Connectivité IP directe** entre les nodes des deux clusters (VPN, peering cloud, ou réseau routé).
2. **CIDR de pods non-chevauchants** entre les clusters (ex: cluster-1 = 10.244.0.0/16, cluster-2 = 10.245.0.0/16) — sinon impossible de distinguer les identités réseau.
3. **CA partagée ou fédérée** entre les instances Cilium des deux clusters, pour que l'identité et le chiffrement (si activé) fonctionnent de façon cohérente cross-cluster.

## 2. CiliumNetworkPolicy cross-cluster
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: frontend-to-backend-cross-cluster
spec:
  endpointSelector:
    matchLabels:
      role: backend
      io.cilium.k8s.policy.cluster: cluster-2
  ingress:
    - fromEndpoints:
        - matchLabels:
            role: frontend
            io.cilium.k8s.policy.cluster: cluster-1
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

## 3. Asymétrie de la policy
Cette policy est appliquée sur les endpoints `backend` (le `endpointSelector` cible `role: backend, cluster: cluster-2`), donc elle contrôle uniquement l'**ingress** vers ces endpoints. Elle n'autorise QUE le sens frontend(cluster-1) → backend(cluster-2). Pour autoriser un trafic retour depuis backend vers frontend, il faudrait une SECONDE policy distincte avec un `endpointSelector` ciblant `frontend` et une règle `ingress` autorisant `backend`.

C'est cohérent avec le modèle de sécurité **identity-based** de Cilium (vu en 4.1) : chaque policy s'applique à un ensemble d'endpoints précis via son `endpointSelector`, et les règles ingress/egress de CETTE policy ne concernent QUE ces endpoints — il n'y a pas de règle bidirectionnelle implicite.

## 4. Commandes de diagnostic Cluster Mesh
```bash
cilium clustermesh status
# Vérifie : l'état de la connexion entre les clusters meshés (connecté/déconnecté),
# le nombre de clusters visibles, la santé du composant clustermesh-apiserver.

cilium clustermesh connectivity
# Teste activement la connectivité réseau ET applicative entre les clusters
# (équivalent d'un "cilium connectivity test" mais étendu au contexte multi-cluster) — utile
# pour isoler si un problème est réseau pur (routage/VPN) ou applicatif (policy, identité).
```

**Point clé pour l'examen** : sache que le label `io.cilium.k8s.policy.cluster` est LA clé pour cibler un cluster spécifique dans une policy — c'est un point de syntaxe précis, probablement testé directement.
