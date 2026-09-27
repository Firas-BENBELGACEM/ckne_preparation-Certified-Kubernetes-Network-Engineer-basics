# Exercice 3.1 — Routage d'inférence avec InferencePool

**Temps cible : 20 minutes**

⚠️ Sujet émergent — cet exercice utilise des serveurs "mock" pour simuler le concept sans nécessiter de vrai LLM/GPU.

## Setup
```bash
# Simule 3 "serveurs d'inférence" avec des temps de réponse différents
kubectl create deployment llm-server --image=nginx --replicas=3
kubectl label pods -l app=llm-server app=llm-server --overwrite
kubectl expose deployment llm-server --port=8000 --target-port=80

# Installer les CRDs Gateway API Inference Extension
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api-inference-extension/releases/latest/download/manifests.yaml
```

## Tâches
1. Explore la doc officielle Gateway API Inference Extension pour identifier la structure exacte de la CRD `InferencePool` dans sa dernière version.
2. Crée un `InferencePool` sélectionnant les pods `llm-server`.
3. Crée un `HTTPRoute` sur `/v1/completions` routant vers cet `InferencePool` (au lieu d'un `backendRef` Service classique).
4. Compare (dans un fichier texte) la différence structurelle entre un `backendRef` pointant vers un Service standard et un `backendRef` pointant vers un `InferencePool`.
5. Explique en 2-3 phrases pourquoi un simple Service Kubernetes ClusterIP ne suffit pas pour ce cas d'usage.
