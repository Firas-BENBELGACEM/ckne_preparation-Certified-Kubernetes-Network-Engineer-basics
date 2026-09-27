# 1.3 Using Linux Tools (iptables, ip, tcpdump) pour les problèmes bas niveau

## Résumé
⚠️ **Nuance importante** : le retour du candidat beta jkroepke indique que dans son expérience, le troubleshooting était "plus piloté par l'UI que prévu" et qu'il n'a pas eu besoin d'`ip` ou `tcpdump`, s'appuyant plutôt sur Prometheus/Jaeger préinstallés. **Mais ce sous-thème figure explicitement dans le blueprint officiel (domaine 1, 15%)** — donc à maîtriser quand même, ne serait-ce que pour la compréhension profonde qui accélère tout le reste.

**Table de correspondance réseau traditionnel ↔ Kubernetes :**

| Cas d'usage | Réseau traditionnel | Kubernetes/Cilium |
|---|---|---|
| Connectivité TCP/IP | ping/traceroute | ping |
| Connectivité HTTP | curl | curl |
| État du réseau | CLI IOS | kubectl / cilium CLI |
| Logs | Syslog | kubectl logs |
| Flux et bande passante | NetFlow/sFlow | Hubble |
| Analyse de trafic | tcpdump/Wireshark | tcpdump/Wireshark/Hubble |
| Génération de trafic | iperf | iperf |

**Commandes essentielles :**
```bash
# Voir les routes et interfaces sur un node
ip route show
ip addr show

# Capturer du trafic depuis l'agent Cilium (DaemonSet)
kubectl exec -n kube-system ds/cilium -- cilium monitor --type drop
kubectl exec -n kube-system ds/cilium -- tcpdump -i any -n port 8472 -c 20   # VXLAN

# Voir les règles iptables injectées (pertinent même avec Cilium en mode legacy/masquerading)
kubectl exec -n kube-system ds/cilium -- iptables -L -n -v | grep -i cilium

# Déployer un conteneur de debug tout-en-un (netshoot)
kubectl run -it debug-pod --image nicolaka/netshoot
kubectl debug <pod-existant> -it --image=nicolaka/netshoot -- tcpdump -i eth0
```

`kubectl debug` est particulièrement utile car il injecte un conteneur éphémère qui **partage le stack réseau** du pod ciblé — donc pas besoin de relancer le pod applicatif.

## Sources officielles
- Cilium troubleshooting : https://docs.cilium.io/en/stable/operations/troubleshooting/
- netshoot : https://github.com/nicolaka/netshoot
