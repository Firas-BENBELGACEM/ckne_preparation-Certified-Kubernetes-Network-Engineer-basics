# Exercice 2.3 — Personnalisation de CoreDNS

**Temps cible : 15 minutes**

## Tâches
1. Édite le Corefile pour ajouter un stub domain : toute requête vers `*.corp.internal` doit être forwardée vers `10.20.0.5`.
2. Redémarre CoreDNS et valide que le nouveau bloc de configuration est bien pris en compte (sans tester la résolution réelle, juste la config).
3. Change le comportement de cache : passe le TTL de cache de 30s à 60s.
4. Crée un pod avec une `dnsPolicy: None` et un `dnsConfig` personnalisé pointant vers `8.8.8.8` comme unique nameserver, puis valide sa résolution DNS externe.
