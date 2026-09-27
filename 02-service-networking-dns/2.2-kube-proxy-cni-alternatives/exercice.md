# Exercice 2.2 — kube-proxy replacement

**Temps cible : 15 minutes**

## Tâches
1. Sur un cluster kind équipé de Cilium (kube-proxy standard actif par défaut), vérifie l'état actuel du kube-proxy replacement.
2. Réinstalle/upgrade Cilium pour activer le mode `kubeProxyReplacement=true`.
3. Vérifie qu'aucun DaemonSet `kube-proxy` classique ne tourne plus (ou qu'il est neutralisé).
4. Déploie un Service ClusterIP et valide que la résolution/le load-balancing fonctionne toujours en mode eBPF pur.
5. Utilise `cilium bpf lb list` pour observer les entrées de load balancing eBPF associées à ton Service.
