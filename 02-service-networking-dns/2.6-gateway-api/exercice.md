# Exercice 2.6 — Gateway API complet

**Temps cible : 25 minutes** (c'est le sujet le plus lourd — accorde-toi plus de temps en entraînement)

## Setup
```bash
# Activer Gateway API support dans Cilium
cilium install --set gatewayAPI.enabled=true --set kubeProxyReplacement=true
cilium status --wait

# Installer les CRDs Gateway API si absentes
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml

kubectl create deployment details --image=nginx --port=9080
kubectl expose deployment details --port=9080
kubectl create deployment productpage --image=httpd --port=9080
kubectl expose deployment productpage --port=9080
```

## Tâches
1. Crée un `GatewayClass` (ou vérifie que celui de Cilium existe déjà) et un `Gateway` nommé `my-gateway` écoutant en HTTP sur le port 80.
2. Crée un `HTTPRoute` routant `/details` vers le Service `details` et `/` (tout le reste) vers `productpage`.
3. Récupère l'IP externe assignée au Gateway (il crée un Service LoadBalancer sous le capot).
4. Teste avec `curl` que `/details` répond bien via le Service `details` et `/` via `productpage`.
5. **Bonus (traffic splitting)** : modifie le HTTPRoute pour envoyer 90% du trafic sur `/` vers `productpage` et 10% vers une version `productpage-v2` (nouveau deployment à créer), en utilisant les poids de `backendRefs`.
6. **Bonus (headers matching)** : ajoute une règle qui route les requêtes contenant le header `magic: foo` vers un backend spécifique.
