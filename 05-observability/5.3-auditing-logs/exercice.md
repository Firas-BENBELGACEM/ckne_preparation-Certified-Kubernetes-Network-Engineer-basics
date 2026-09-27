# Exercice 5.3 — Audit de trafic via logs

**Temps cible : 15 minutes**

## Setup
```bash
kubectl create deployment web --image=nginx --replicas=2
kubectl expose deployment web --port=80
kubectl run client --image=nicolaka/netshoot -- sleep 3600
kubectl run attacker --image=nicolaka/netshoot -- sleep 3600

kubectl apply -f - <<POLICY
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-client
spec:
  podSelector:
    matchLabels: {app: web}
  policyTypes: ["Ingress"]
  ingress:
    - from:
        - podSelector:
            matchLabels: {run: client}
POLICY

# Génère du trafic normal et du trafic rejeté
for i in $(seq 1 5); do kubectl exec client -- curl -m 1 -s -o /dev/null http://web/; done
for i in $(seq 1 5); do kubectl exec attacker -- curl -m 1 -s -o /dev/null http://web/; done
```

## Tâches
1. Utilise `hubble observe` pour lister tous les flux vers `web` avec leur verdict.
2. Filtre spécifiquement les flux `DROPPED` et identifie leur source.
3. Filtre spécifiquement les flux provenant de `attacker` — confirme qu'ils sont tous rejetés.
4. Sans modifier la policy, produis un résumé texte (2-3 lignes) de ce que tu observerais dans un vrai log d'audit si `attacker` tentait 100 requêtes en 10 secondes (à quoi ressemblerait un "signal suspect" dans les logs, versus un simple trafic normalement rejeté) ?
