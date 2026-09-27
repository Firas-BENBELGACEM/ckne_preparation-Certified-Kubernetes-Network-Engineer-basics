# Exercice 3.3 — Egress Gateway

**Temps cible : 15 minutes**

## Contexte
L'équipe sécurité exige que tout le trafic sortant des pods labellisés `team=payments` dans le namespace `default` sorte du cluster avec une IP source fixe `10.168.60.200`, via le node `kind-worker`.

## Tâches
1. Labellise `kind-worker` comme node egress gateway.
2. Crée la `CiliumEgressGatewayPolicy` correspondant à l'exigence.
3. Déploie un pod labellisé `team=payments` et vérifie (via un service externe de test comme `ifconfig.me` simulé, ou via capture tcpdump sur le node egress) que son trafic sortant utilise bien l'IP configurée.
4. Vérifie qu'un pod SANS le label `team=payments` n'est PAS affecté par cette politique (son IP source normale reste utilisée).
