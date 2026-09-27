# Correction 3.1

```bash
# 1. Consulter systématiquement la doc au moment de l'examen — les CRD de ce projet
# évoluent vite (alpha), donc TOUJOURS vérifier la version exacte plutôt que mémoriser :
# https://gateway-api-inference-extension.sigs.k8s.io/api-types/inferencepool/

# 2. InferencePool
kubectl apply -f - <<EOF2
apiVersion: inference.networking.k8s.io/v1alpha2
kind: InferencePool
metadata:
  name: llm-pool
spec:
  selector:
    app: llm-server
  targetPortNumber: 80
EOF2

kubectl get inferencepool llm-pool

# 3. HTTPRoute vers l'InferencePool
kubectl apply -f - <<EOF2
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
EOF2

# 4. Différence structurelle :
# backendRef vers Service classique :
#   backendRefs:
#     - name: my-service
#       port: 80
#   (pas de "group" ni "kind" explicites car Service = ressource core par défaut)
#
# backendRef vers InferencePool :
#   backendRefs:
#     - group: inference.networking.k8s.io
#       kind: InferencePool
#       name: llm-pool
#   (group + kind OBLIGATOIRES car ce n'est plus une ressource core Kubernetes,
#    HTTPRoute doit savoir résoudre un type de ressource personnalisé)
```

**Réponse à la question 5** : un Service ClusterIP classique fait du round-robin ou du load balancing purement basé sur la connexion (nombre de connexions actives), sans aucune connaissance de la charge RÉELLE du modèle chargé sur chaque pod (longueur de la queue d'inférence, utilisation du KV-cache GPU, latence de génération en cours). Un `InferencePool` délègue le choix du backend à un Endpoint Picker qui interroge ces métriques applicatives spécifiques à l'inférence, permettant un routage qui évite de surcharger un pod déjà occupé sur une génération longue.

**Point clé pour l'examen** : ce sujet étant en alpha et évolutif, la compétence testée est probablement moins "connaître la syntaxe exacte par cœur" que "savoir naviguer la doc rapidement et comprendre le PROBLÈME que résout cette extension" — exactement l'esprit relevé par le retour de beta ("documentation matters more than memorization").
