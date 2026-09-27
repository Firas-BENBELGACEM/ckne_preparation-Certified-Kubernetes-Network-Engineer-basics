# Exercice 1.1 — Installation et remplacement de CNI

**Temps cible : 20 minutes**

## Contexte
Un cluster kind existe déjà avec **Flannel** comme CNI. Ton manager veut migrer vers Cilium sans reconstruire le cluster. Une interruption de service est acceptée.

## Setup de départ
```bash
cat <<CFG | kind create cluster --name ckne-ex11 --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: "10.244.0.0/16"
nodes:
  - role: control-plane
  - role: worker
  - role: worker
CFG

kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
kubectl create deployment web --image=nginx --replicas=3
kubectl expose deployment web --port=80
```

## Tâches
1. Vérifie que Flannel tourne et que les pods `web` ont bien une IP.
2. Désinstalle proprement Flannel.
3. Installe Cilium avec le Cilium CLI, en forçant explicitement `--set ipam.mode=cluster-pool` et `--set ipam.operator.clusterPoolIPv4PodCIDRList=10.244.0.0/16` (garder la cohérence avec le podSubnet du cluster).
4. Fais en sorte que les pods `web` existants récupèrent une IP gérée par Cilium.
5. Valide la connectivité de bout en bout avec `cilium connectivity test`.
6. Documente (dans un fichier texte) les 3 commandes que tu as utilisées pour nettoyer les résidus de Flannel.
