# 1.1 Installing and Configuring CNI Plugins

## Résumé
Kubernetes ne fournit pas de mise en réseau par défaut : il délègue cela à un plugin **CNI (Container Network Interface)**, une spécification (pas une implémentation) qui définit comment un runtime de conteneurs doit configurer la mise en réseau des pods. Pense-y comme à une RFC IETF que différents vendeurs (Cilium, Calico, Flannel...) implémentent chacun à leur façon.

Le modèle réseau Kubernetes impose 3 règles fondamentales :
- Chaque pod reçoit une IP unique à l'échelle du cluster
- Les pods peuvent communiquer entre eux sans NAT
- Les agents du node (kubelet) peuvent communiquer avec tous les pods du node

**Cilium** est le CNI recommandé pour la CKNE (seul projet CNCF Graduated dans la catégorie networking cloud-native). Installation via son CLI dédié :

```bash
cilium install --version 1.16.0
cilium status --wait
cilium connectivity test
```

Le CLI Cilium traduit ses flags en valeurs Helm sous le capot — comprendre cette correspondance est explicitement cité comme un point à maîtriser par un candidat de la beta. Compare toujours :
```bash
cilium install --help
helm show values cilium/cilium
```

**Remplacement de CNI (Flannel → Cilium)** — un candidat beta rapporte avoir eu cet exact scénario, avec perturbation de service autorisée. La procédure générale :
1. Supprimer les manifests Flannel (`kubectl delete -f flannel.yaml` ou équivalent)
2. Nettoyer les interfaces réseau résiduelles sur chaque node si besoin (`ip link delete flannel.1`, `ip link delete cni0`)
3. Installer Cilium avec `cilium install`
4. Redémarrer les pods existants pour qu'ils reçoivent une IP gérée par Cilium (`kubectl rollout restart` ou suppression des pods)
5. Valider avec `cilium status --wait` et `cilium connectivity test`

## Sources officielles
- Kubernetes Networking Model : https://kubernetes.io/docs/concepts/cluster-administration/networking/
- CNI Specification : https://github.com/containernetworking/cni/blob/main/SPEC.md
- Cilium Quick Install : https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/
- Cilium Helm Reference : https://docs.cilium.io/en/stable/helm-reference/
