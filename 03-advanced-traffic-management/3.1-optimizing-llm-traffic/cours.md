# 3.1 Optimizing LLM Traffic

## Résumé
⚠️ **Sujet le plus récent et le moins documenté du blueprint.** Confirmé explicitement par les deux retours de beta comme abordé ("inference avec Gateway" et "InferencePool / Gateway API Inference Extension"). Peu de ressources existent encore — ce cours synthétise le maximum d'info disponible à date.

**Le problème que ça résout** : le load balancing L4/L7 classique de Kubernetes (round-robin, least-connections) est **inadapté au trafic d'inférence LLM**, car :
- Les requêtes ont des temps de traitement très variables (un prompt court ≠ une génération longue)
- Le "coût" d'une requête dépend du modèle chargé sur chaque replica, de l'état du KV-cache GPU, de la longueur du contexte
- Un routage naïf peut envoyer une requête vers un pod déjà saturé alors qu'un autre est libre

**Gateway API Inference Extension** ajoute une nouvelle CRD : **`InferencePool`** — un pool de backends spécialisés pour l'inférence, avec un routage conscient de la charge réelle (métriques custom exposées par le serveur d'inférence, ex: longueur de queue, utilisation KV-cache) plutôt qu'un simple round-robin.

```yaml
apiVersion: inference.networking.k8s.io/v1alpha2
kind: InferencePool
metadata:
  name: llm-pool
spec:
  selector:
    app: llm-server
  targetPortNumber: 8000
  extensionRef:
    name: llm-endpoint-picker
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: llm-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /v1/completions
      backendRefs:
        - group: inference.networking.k8s.io
          kind: InferencePool
          name: llm-pool
```

Le routage passe par un **Endpoint Picker (EPP)** — un composant qui interroge les métriques de charge réelles des serveurs d'inférence (souvent exposées par le serveur de modèle, ex: vLLM) pour choisir dynamiquement le meilleur backend, contrairement à un Service Kubernetes classique qui ne fait que du load balancing aveugle.

## Sources officielles
- Gateway API Inference Extension : https://gateway-api-inference-extension.sigs.k8s.io/
- InferencePool API reference : https://gateway-api-inference-extension.sigs.k8s.io/api-types/inferencepool/
