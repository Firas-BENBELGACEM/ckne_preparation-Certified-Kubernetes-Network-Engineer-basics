# Exercice 5.2 — Diagnostic de latence avec Jaeger

**Temps cible : 20 minutes**

## Setup
```bash
istioctl install --set profile=demo --set meshConfig.defaultConfig.tracing.sampling=100.0 -y
kubectl label namespace default istio-injection=enabled --overwrite
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/jaeger.yaml

# Déploie une chaîne de 3 services, dont un artificiellement lent
kubectl create deployment service-a --image=nginx
kubectl create deployment service-b --image=nginx
kubectl create deployment service-c --image=httpd   # simule un service "lent" (autre image = latence de démarrage différente)
kubectl expose deployment service-a --port=80
kubectl expose deployment service-b --port=80
kubectl expose deployment service-c --port=80
kubectl rollout restart deployment service-a service-b service-c
```

## Tâches
1. Accède à l'UI Jaeger via port-forward.
2. Génère quelques requêtes traversant `service-a` → `service-b` → `service-c` (simulé via curl direct pour cet exercice simplifié).
3. Recherche les traces récentes du service concerné dans Jaeger.
4. Ouvre une trace et identifie quel span concentre le plus de temps.
5. Explique en 2 phrases comment tu distinguerais, à partir d'une trace Jaeger, une lenteur due au RÉSEAU (latence entre spans) d'une lenteur due à l'APPLICATIF (durée du span lui-même).
