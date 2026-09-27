# Exercice 1.3 — Diagnostic bas niveau

**Temps cible : 15 minutes**

## Contexte
Deux pods sur des nodes différents ne semblent pas communiquer. Tu dois diagnostiquer avec les outils Linux bas niveau, sans regarder les logs applicatifs.

## Setup
```bash
kubectl create deployment client --image=nicolaka/netshoot -- sleep 3600
kubectl create deployment server --image=nginx
kubectl expose deployment server --port=80
```

## Tâches
1. Identifie sur quels nodes tournent `client` et `server`.
2. Depuis le pod `client`, tente une connexion vers `server` (curl) et note le résultat.
3. Utilise `cilium monitor --type drop` pour observer si des paquets sont droppés.
4. Utilise `tcpdump` sur l'agent Cilium pour capturer le trafic VXLAN (port 8472) pendant que tu relances la requête.
5. Utilise `kubectl debug` pour injecter un conteneur de debug dans le pod `server` et vérifier que le process nginx écoute bien sur le bon port avec `ss -tlnp`.
