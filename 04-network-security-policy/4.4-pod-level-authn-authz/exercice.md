# Exercice 4.4 — PeerAuthentication et AuthorizationPolicy Istio

**Temps cible : 25 minutes**

## Setup
```bash
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled --overwrite

kubectl create deployment frontend --image=nginx
kubectl create deployment backend --image=nginx
kubectl expose deployment backend --port=80
kubectl rollout restart deployment frontend backend   # pour injecter le sidecar istio-proxy
```

## Tâches
1. Applique une `PeerAuthentication` en mode `STRICT` pour le namespace `default` (force mTLS partout).
2. Valide que la communication frontend → backend fonctionne toujours (le sidecar gère le mTLS automatiquement, transparent pour l'appli).
3. Ajoute une `AuthorizationPolicy` sur `backend` qui n'autorise QUE les requêtes GET provenant du service account `frontend`.
4. Teste avec une requête `POST` depuis `frontend` (doit être rejetée avec 403).
5. Crée un troisième pod `mallory` SANS lui donner le bon service account, et vérifie qu'il ne peut ni faire GET ni POST vers `backend`.
6. Explique en une phrase la différence de comportement entre une requête bloquée par `PeerAuthentication` (identité non vérifiable) et une requête bloquée par `AuthorizationPolicy` (identité vérifiée mais action non permise).
