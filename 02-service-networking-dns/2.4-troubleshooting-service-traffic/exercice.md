# Exercice 2.4 — Troubleshooting méthodique de Service

**Temps cible : 15 minutes**

## Contexte
Un Service `checkout` ne répond plus. Trois pannes distinctes sont possibles — identifie et corrige **chacune** dans l'ordre où tu les rencontres.

## Setup (contient volontairement plusieurs erreurs)
```bash
kubectl create deployment checkout --image=nginx --replicas=3
kubectl label pods -l app=checkout tier=backend --overwrite
kubectl apply -f - <<POLICY
apiVersion: v1
kind: Service
metadata:
  name: checkout
spec:
  selector:
    app: checkout-svc    # <- selector volontairement erroné
  ports:
    - port: 80
      targetPort: 80
POLICY
```

## Tâches
1. Diagnostique pourquoi le Service `checkout` n'a aucun endpoint.
2. Corrige le selector.
3. Une fois corrigé, ajoute une NetworkPolicy `deny-all` par erreur dans le namespace et observe l'impact — puis corrige en autorisant explicitement le trafic nécessaire.
4. Valide de bout en bout avec un pod client.
