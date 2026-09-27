# 1.4 Troubleshooting Pod Connectivity (DNS, pod-to-pod)

## Résumé
Deux grandes familles de pannes de connectivité pod à traiter en boucle jusqu'à réflexe :

### A. Connectivité pod-to-pod
Checklist de diagnostic dans l'ordre :
1. Les deux pods sont-ils `Running` et `Ready` ? `kubectl get pods -o wide`
2. Ont-ils bien reçu une IP dans le bon PodCIDR ?
3. Une NetworkPolicy bloque-t-elle le trafic ? `kubectl get networkpolicy,ciliumnetworkpolicy -A`
4. Le CNI est-il en bonne santé sur les deux nodes concernés ? `cilium status`
5. Le trafic cross-node passe-t-il par l'overlay (VXLAN) ou en direct routing ? Voir domaine 1.1.

### B. Résolution DNS
Kubernetes utilise CoreDNS par défaut. Format des noms :
- Pod : `10-244-1-234.default.pod.cluster.local` (IP avec tirets)
- Service : `my-service.my-namespace.svc.cluster.local`

Policy DNS par défaut : `ClusterFirst` — toute requête ne correspondant pas au suffixe du cluster est transférée à un nameserver upstream.

```bash
# Test DNS depuis un pod
kubectl run -it dnsutils --image=registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3 -- nslookup my-service.default.svc.cluster.local

# Vérifier l'état de CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Vérifier la résolution depuis l'intérieur même de CoreDNS
kubectl exec -n kube-system deploy/coredns -- cat /etc/coredns/Corefile
```

Citation utile à retenir : **"DNS is often the culprit in Kubernetes, too"** — si un candidat bloque sur un problème de connectivité qui semble aléatoire ou intermittent, vérifier DNS avant tout le reste.

## Sources officielles
- DNS for Services and Pods : https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
- Debug DNS Resolution : https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
