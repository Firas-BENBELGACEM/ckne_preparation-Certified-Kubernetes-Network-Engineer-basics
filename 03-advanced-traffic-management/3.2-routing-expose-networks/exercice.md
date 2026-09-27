# Exercice 3.2 — Exposition réseau via BGP et L2

**Temps cible : 20 minutes**

## Tâches — Partie A (BGP)
1. Configure une `CiliumBGPPeeringPolicy` avec un ASN local 65010, peering vers un voisin fictif `10.0.0.1` en ASN 65000.
2. Vérifie l'état de la session avec `cilium bgp peers`.
3. Configure le pool LB-IPAM pour que Cilium assigne des IP externes aux Services LoadBalancer, puis vérifie que ces IP sont bien annoncées en BGP (`exportPodCIDR` ou announces de Service).

## Tâches — Partie B (L2/ARP)
4. Configure une `CiliumL2AnnouncementPolicy` qui répond aux ARP uniquement pour les Services dans le namespace `default`, sur l'interface `eth0`.
5. Crée un Service LoadBalancer et vérifie (via `arping` ou capture tcpdump) que Cilium répond bien aux requêtes ARP pour son IP.
