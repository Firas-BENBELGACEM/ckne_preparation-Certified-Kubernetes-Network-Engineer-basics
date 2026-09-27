# Exercice 4.3 — TLS sur Gateway API avec cert-manager

**Temps cible : 20 minutes**

## Setup
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl wait --for=condition=Available --timeout=120s -n cert-manager deployment --all
```

## Tâches
1. Crée un `ClusterIssuer` self-signed.
2. Crée un `Certificate` pour le domaine `app.ckne.local`, stocké dans un Secret `app-tls`.
3. Vérifie que le certificat est bien émis (`READY=True`) et que le Secret existe.
4. Crée un `Gateway` avec un listener HTTPS sur le port 443 référençant ce Secret.
5. **Panne volontaire** : modifie le `certificateRefs` du Gateway pour pointer vers un nom de Secret inexistant (`wrong-secret-name`) et observe l'état du Gateway (`kubectl describe gateway`).
6. Corrige et revalide.
