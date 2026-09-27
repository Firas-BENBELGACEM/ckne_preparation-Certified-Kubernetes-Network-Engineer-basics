# 2.3 Customizing coreDNS for Services

## Résumé
CoreDNS est déployé par défaut comme Deployment dans `kube-system`, configuré via une ConfigMap appelée `coredns` contenant le **Corefile** (syntaxe de plugin-chain façon nginx.conf).

**Structure type d'un Corefile :**
```
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
    }
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

**Personnalisations fréquentes à l'examen :**

1. **Ajouter un domaine de résolution personnalisé (stub domain)** — rediriger les requêtes pour `internal.company.com` vers un serveur DNS interne spécifique :
```
internal.company.com:53 {
    forward . 10.10.0.53
}
```

2. **Changer le forwarder upstream par défaut** (au lieu de `/etc/resolv.conf`) :
```
forward . 8.8.8.8 8.8.4.4
```

3. **DNS policy par pod** — `ClusterFirst` (défaut), `Default`, `None`, `ClusterFirstWithHostNet`, configurable via `dnsPolicy` dans le spec du pod, avec possibilité d'override fin via `dnsConfig`.

**Après modification du Corefile, il FAUT relancer CoreDNS pour appliquer :**
```bash
kubectl edit configmap coredns -n kube-system
kubectl rollout restart deployment coredns -n kube-system
```

## Sources officielles
- Customizing DNS Service : https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/
- CoreDNS plugins reference : https://coredns.io/manual/toc/
