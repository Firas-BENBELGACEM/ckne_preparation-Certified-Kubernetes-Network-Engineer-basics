# 2.2 Understanding kube-proxy and CNI Alternatives

## Résumé
`kube-proxy` **n'est pas un vrai proxy** — c'est un composant qui surveille le control plane Kubernetes et maintient à jour les règles de filtrage de paquets du système d'exploitation (historiquement `iptables`, plus récemment `ipvs`) pour router le trafic des Services vers les bons pods.

**Le problème d'iptables** : c'est une technologie vieille de 25 ans reposant sur un algorithme **séquentiel** — chaque paquet traverse les règles une par une jusqu'à trouver une correspondance. Le temps de traitement croît linéairement avec le nombre de règles = dégradation de performance à l'échelle (nombreux Services/endpoints).

**L'alternative eBPF (Cilium kube-proxy replacement)** : remplace iptables par des **tables de hachage eBPF**, offrant un accès quasi O(1) peu importe le nombre de Services. Le gain de performance devient flagrant à partir de centaines/milliers de Services.

```bash
# Installer Cilium en mode kube-proxy replacement (remplace kube-proxy entièrement)
cilium install --set kubeProxyReplacement=true

# Vérifier le mode actif
cilium status | grep KubeProxyReplacement
kubectl exec -n kube-system ds/cilium -- cilium status --verbose | grep -A5 "KubeProxyReplacement"
```

**Table de décision rapide pour l'examen :**
| Besoin | Solution |
|---|---|
| Petit cluster, compatibilité maximale | kube-proxy (iptables) |
| Cluster moyen, meilleure perf sans tout changer | kube-proxy (mode ipvs) |
| Grande échelle, perf maximale, observabilité fine | Cilium eBPF kube-proxy replacement |

## Sources officielles
- kube-proxy : https://kubernetes.io/docs/reference/networking/virtual-ips/
- Cilium kube-proxy replacement : https://docs.cilium.io/en/stable/network/kubernetes/kubeproxy-free/
