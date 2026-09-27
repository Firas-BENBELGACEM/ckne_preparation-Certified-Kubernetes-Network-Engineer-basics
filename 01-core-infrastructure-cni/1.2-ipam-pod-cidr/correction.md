# Correction 1.2

```bash
# 1. Vérifier le mode IPAM
kubectl get -n kube-system configmap cilium-config -o yaml | grep -i ipam
cilium config view | grep -i ipam

# 2. Ajouter un pool supplémentaire (nécessite le mode cluster-scope ou multi-pool)
# En mode cluster-scope, on étend via l'option clusterPoolIPv4PodCIDRList au niveau Helm :
cilium upgrade --set ipam.operator.clusterPoolIPv4PodCIDRList="{10.244.0.0/16,10.245.0.0/16}"

# En mode multi-pool, on créerait plutôt un objet CiliumPodIPPool dédié :
cat <<EOF2 | kubectl apply -f -
apiVersion: cilium.io/v2alpha1
kind: CiliumPodIPPool
metadata:
  name: extra-pool
spec:
  ipv4:
    cidrs:
      - "10.245.0.0/16"
    maskSize: 24
EOF2

# 3. Déployer un pod et vérifier son IP
kubectl run test-ipam --image=nginx
kubectl get pod test-ipam -o jsonpath='{.status.podIP}'

# 4. Explication :
# En mode "kubernetes-host-scope", le découpage des sous-réseaux par node se fait à partir
# d'UN SEUL préfixe cluster-wide fixé au démarrage. Il n'existe aucun mécanisme pour ajouter
# un second préfixe une fois le cluster créé — contrairement au mode cluster-scope (Cilium
# gère nativement l'ajout de préfixes) ou multi-pool (pools multiples et dynamiques).
```

**Point clé pour l'examen** : sache identifier RAPIDEMENT quel mode IPAM est actif (`cilium config view | grep ipam`) avant de choisir la bonne méthode d'extension — c'est un piège classique de vouloir appliquer une solution multi-pool alors que le cluster est en cluster-scope, ou inversement.
