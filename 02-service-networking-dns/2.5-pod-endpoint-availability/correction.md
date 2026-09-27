# Correction 2.5

```bash
# 1. Ajout d'un readinessProbe invalide
kubectl patch deployment flaky -p '{
  "spec": {"template": {"spec": {"containers": [{
    "name": "nginx",
    "readinessProbe": {
      "httpGet": {"path": "/nonexistent-path", "port": 80},
      "periodSeconds": 5,
      "failureThreshold": 2
    }
  }]}}}}'

# 2. Observation
kubectl get pods -l app=flaky   # colonne READY passe à 0/1 après quelques secondes
kubectl get endpointslices -l kubernetes.io/service-name=flaky -o yaml   # endpoints list vide ou "Ready: false"

# 3. Correction du path
kubectl patch deployment flaky -p '{
  "spec": {"template": {"spec": {"containers": [{
    "name": "nginx",
    "readinessProbe": {
      "httpGet": {"path": "/", "port": 80},
      "periodSeconds": 5,
      "failureThreshold": 2
    }
  }]}}}}'
kubectl get pods -l app=flaky   # READY 1/1 après stabilisation

# 4. PodDisruptionBudget
kubectl apply -f - <<EOF2
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: flaky-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: flaky
EOF2

# 5. Test du PDB
kubectl get pdb flaky-pdb   # colonne ALLOWED DISRUPTIONS doit indiquer 1 (3 replicas - minAvailable 2)
kubectl delete pod -l app=flaky --field-selector=status.phase=Running  # tentative de tout supprimer
# -> le PDB protège: l'API refusera une éviction (via eviction API, pas delete direct) qui violerait minAvailable
```

**Point clé pour l'examen** : `kubectl delete pod` direct N'EST PAS bloqué par un PodDisruptionBudget — seule l'API d'éviction (`kubectl drain`, ou un appel `POST /eviction`) respecte le PDB. Ne pas confondre les deux mécanismes à l'examen si la question porte spécifiquement sur les évictions volontaires.
