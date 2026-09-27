# Exercice 2.1 — Services L4

**Temps cible : 15 minutes**

## Setup
```bash
kubectl create deployment echoserver --image=k8s.gcr.io/echoserver:1.10 --port=8080 --replicas=3
```

## Tâches
1. Crée un Service `ClusterIP` nommé `echoserver` exposant le port 80 → targetPort 8080.
2. Volontairement, un collègue a modifié le Service avec `targetPort: 8081` (mauvais port). Reproduis cette panne, diagnostique-la avec `kubectl` (sans regarder les events), puis corrige.
3. Crée un second Service de type `NodePort` pour le même deployment, et récupère le port externe alloué.
4. Depuis l'extérieur du cluster (ou depuis un node), teste la connectivité NodePort avec `curl`.
5. **Cas avancé (Service sans selector)** : crée un Service `external-api` SANS selector pointant vers une IP externe fictive `192.0.2.10:443`, en créant manuellement l'EndpointSlice associé.
