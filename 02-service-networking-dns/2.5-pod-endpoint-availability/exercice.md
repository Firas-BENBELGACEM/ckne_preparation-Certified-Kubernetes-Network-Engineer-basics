# Exercice 2.5 — Disponibilité d'endpoint et probes

**Temps cible : 15 minutes**

## Setup
```bash
kubectl create deployment flaky --image=nginx --replicas=3
kubectl expose deployment flaky --port=80
```

## Tâches
1. Ajoute un `readinessProbe` HTTP sur `/nonexistent-path` (volontairement invalide) à ce deployment.
2. Observe l'impact sur les EndpointSlices du Service `flaky`.
3. Corrige le `readinessProbe` pour pointer vers `/` (qui répond bien sur nginx).
4. Ajoute un `PodDisruptionBudget` garantissant qu'au moins 2 pods sur 3 restent disponibles en permanence.
5. Teste ce PDB en tentant un `kubectl drain` simulé (ou en supprimant volontairement 2 pods d'un coup) et observe le comportement.
