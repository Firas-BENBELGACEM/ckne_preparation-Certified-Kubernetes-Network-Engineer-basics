# Correction 4.2

```bash
# 1. État initial
cilium status | grep -i encryption

# 2. Activation WireGuard
cilium upgrade --set encryption.enabled=true --set encryption.type=wireguard
cilium status --wait

# 3. Vérification
cilium status | grep -A3 Encryption
kubectl exec -n kube-system ds/cilium -- cilium encrypt status

# 4. Placement forcé sur nodes différents
kubectl run pod-a --image=nicolaka/netshoot --overrides='{"spec":{"nodeName":"kind-worker"}}' -- sleep 3600
kubectl run pod-b --image=nginx --overrides='{"spec":{"nodeName":"kind-worker2"}}'

POD_B_IP=$(kubectl get pod pod-b -o jsonpath='{.status.podIP}')
kubectl exec pod-a -- curl -m 2 -s "http://${POD_B_IP}"

# Capture sur l'interface PHYSIQUE du node (via l'agent Cilium, pas dans le pod) :
kubectl exec -n kube-system ds/cilium -- tcpdump -i eth0 -n host "$POD_B_IP" -c 10 -X
# Le contenu applicatif (HTTP headers, body) ne doit PAS être lisible en clair dans le dump
```

**Réponse à la question 5** : le chiffrement WireGuard/IPsec de Cilium opère au niveau du **tunnel entre nodes** (encapsulation sur l'interface physique/overlay), PAS au niveau de l'interface virtuelle du pod (`lxc*`/`eth0` dans le namespace réseau du pod). Une capture DANS le pod se fait donc AVANT le chiffrement (côté émission) ou APRÈS déchiffrement (côté réception) — le trafic y apparaît donc naturellement en clair, ce qui est normal et attendu, pas un signe de panne.

**Point clé pour l'examen** : ne pas confondre "le chiffrement ne fonctionne pas" avec "je capture au mauvais endroit du pipeline" — c'est un piège de diagnostic classique.
