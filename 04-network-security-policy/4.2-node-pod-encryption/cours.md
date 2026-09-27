# 4.2 Implementing Node and Pod Level Encryption

## Résumé
Chiffrer le trafic en transit entre pods situés sur des nodes différents (ou entre clusters) — pertinent surtout sur infra cloud partagée où le réseau physique n'est pas sous contrôle total. Cilium propose 2 méthodes via son **Transparent Encryption** :

| Méthode | Contrôle | Complexité |
|---|---|---|
| **IPsec** | Choix libre de l'algorithme, du cipher, gestion des clés | Plus flexible mais plus complexe à opérer |
| **WireGuard** | Opinionated — crypto robuste fixée, pas de choix de cipher | Plus simple à activer, moins de surface de config |

**Activer WireGuard (le plus simple à mettre en œuvre à l'examen) :**
```bash
cilium install --set encryption.enabled=true --set encryption.type=wireguard
cilium status | grep Encryption
```

**Activer IPsec :**
```bash
# Nécessite un secret Kubernetes contenant la clé
kubectl create -n kube-system secret generic cilium-ipsec-keys \
  --from-literal=keys="3 rfc4106(gcm(aes)) $(echo $(dd if=/dev/urandom count=20 bs=1 2>/dev/null | xxd -p -c 64)) 128"

cilium install --set encryption.enabled=true --set encryption.type=ipsec
```

**Important** : les deux méthodes ne chiffrent le trafic QUE si les pods sont sur des **nodes différents**. Le trafic intra-node (même machine) transite localement et n'est pas chiffré par ce mécanisme (ce n'est généralement pas un problème car ce trafic ne quitte jamais la mémoire/le kernel du node).

**Cluster Mesh et chiffrement** : les mêmes mécanismes (IPsec/WireGuard) peuvent chiffrer le trafic ENTRE clusters dans une topologie multi-cluster, pas seulement entre nodes d'un même cluster.

## Sources officielles
- Cilium Transparent Encryption : https://docs.cilium.io/en/stable/security/network/encryption/
- Cilium WireGuard : https://docs.cilium.io/en/stable/security/network/encryption-wireguard/
- Cilium IPsec : https://docs.cilium.io/en/stable/security/network/encryption-ipsec/
