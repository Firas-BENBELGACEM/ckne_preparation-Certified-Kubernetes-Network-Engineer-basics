# Exercice 4.1 — Network Policies L3/L4 et L7

**Temps cible : 25 minutes**

## Setup
```bash
kubectl create namespace shop
kubectl create deployment frontend -n shop --image=nginx --labels="org=empire,class=frontend"
kubectl create deployment deathstar -n shop --image=nginx --labels="org=empire,class=deathstar"
kubectl expose deployment deathstar -n shop --port=80
kubectl create deployment mallory -n shop --image=nginx --labels="org=rebels"
```

## Tâches
1. Écris une `NetworkPolicy` standard qui n'autorise l'ingress vers `deathstar` QUE depuis les pods labellisés `org=empire`, sur le port 80.
2. Valide que `frontend` (org=empire) peut joindre `deathstar`, mais que `mallory` (org=rebels) ne le peut pas.
3. Remplace cette policy par une `CiliumNetworkPolicy` L7 : autorise UNIQUEMENT les requêtes HTTP `POST` sur `/v1/request-landing` depuis `org=empire` — tout le reste (autres méthodes, autres paths) doit être rejeté même venant de `frontend`.
4. Valide avec `curl -X GET` (doit échouer) puis `curl -X POST http://deathstar/v1/request-landing` (doit réussir) depuis `frontend`.
5. Passe le mode d'enforcement Cilium en `never` (audit-only) et observe si le trafic qui était bloqué passe maintenant.
