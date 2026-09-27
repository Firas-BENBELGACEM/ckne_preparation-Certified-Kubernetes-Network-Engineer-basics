# Exercice 1.2 — IPAM et Pod CIDR

**Temps cible : 15 minutes**

## Contexte
Ton cluster Cilium tourne en mode `cluster-pool` avec un CIDR initial `10.244.0.0/16`. L'équipe plateforme signale que le cluster va bientôt manquer d'IP disponibles pour les pods car de nouveaux nodes sont ajoutés fréquemment.

## Tâches
1. Vérifie le mode IPAM actuellement configuré et le(s) CIDR alloué(s).
2. Ajoute un pool IPAM supplémentaire `10.245.0.0/16` sans casser les pods existants.
3. Déploie un nouveau pod et vérifie de quel pool il reçoit son IP.
4. Explique (dans un commentaire) pourquoi le mode `kubernetes-host-scope` n'aurait pas permis cette opération.
