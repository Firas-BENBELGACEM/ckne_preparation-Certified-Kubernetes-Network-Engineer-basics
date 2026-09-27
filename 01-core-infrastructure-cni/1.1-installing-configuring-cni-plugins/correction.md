# Correction 1.1

```bash
# 1. Vérification de l'état initial
kubectl get pods -n kube-flannel
kubectl get pods -o wide

# 2. Désinstallation de Flannel
kubectl delete -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Nettoyage des interfaces résiduelles sur CHAQUE node (via docker exec car kind = nodes en conteneurs)
for node in $(kind get nodes --name ckne-ex11); do
  docker exec "$node" ip link delete flannel.1 2>/dev/null || true
  docker exec "$node" ip link delete cni0 2>/dev/null || true
  docker exec "$node" rm -rf /etc/cni/net.d/*flannel* 2>/dev/null || true
done

# 3. Installation de Cilium avec IPAM cohérent
cilium install --version 1.16.0 \
  --set ipam.mode=cluster-pool \
  --set ipam.operator.clusterPoolIPv4PodCIDRList=10.244.0.0/16

cilium status --wait

# 4. Forcer les pods existants à récupérer une IP Cilium
kubectl rollout restart deployment web
kubectl get pods -o wide   # vérifier que les nouvelles IP sont dans 10.244.0.0/16 et gérées par Cilium

# 5. Validation
cilium connectivity test

# 6. Commandes clés de nettoyage documentées :
# - kubectl delete -f <manifest flannel>
# - ip link delete flannel.1 / cni0 sur chaque node
# - rm -rf /etc/cni/net.d/*flannel*
```

**Points de vigilance pour l'examen :**
- Si le podSubnet du cluster kind ne correspond pas à l'IPAM de Cilium, les nouveaux pods peuvent rester en `Pending` ou `ContainerCreating` — toujours vérifier la cohérence.
- `cilium status --wait` bloque tant que tous les composants Cilium (agent, operator, envoy si activé) ne sont pas `Ready`.
- Ne pas oublier de relancer les pods applicatifs existants : Cilium ne migre pas rétroactivement les pods déjà démarrés sous l'ancien CNI.
