# Correction 1.3

```bash
# 1. Localisation des pods
kubectl get pods -o wide

# 2. Test de connectivité
CLIENT_POD=$(kubectl get pod -l app=client -o jsonpath='{.items[0].metadata.name}')
kubectl exec "$CLIENT_POD" -- curl -m 2 -sv http://server

# 3. Observation des drops Cilium (à lancer dans un terminal séparé, en parallèle du test)
kubectl exec -n kube-system ds/cilium -- cilium monitor --type drop

# 4. Capture VXLAN pendant le test
kubectl exec -n kube-system ds/cilium -- tcpdump -i any -n port 8472 -c 20 &
kubectl exec "$CLIENT_POD" -- curl -m 2 -sv http://server

# 5. Debug du pod server
SERVER_POD=$(kubectl get pod -l app=server -o jsonpath='{.items[0].metadata.name}')
kubectl debug "$SERVER_POD" -it --image=nicolaka/netshoot -- ss -tlnp
```

**Grille de lecture des résultats :**
- Si `cilium monitor --type drop` montre des paquets DROPPED avec une raison `Policy denied` → c'est une NetworkPolicy qui bloque, pas un problème CNI (voir domaine 4.1).
- Si le tcpdump VXLAN ne montre AUCUN paquet alors que le client a bien émis la requête → problème de routage/overlay entre nodes (vérifier `cilium status`, l'état du tunnel).
- Si `ss -tlnp` sur le pod server ne montre rien sur le port 80 → le problème n'est pas réseau du tout, c'est applicatif (mauvais targetPort côté Service, cf. domaine 2.1).

Cette démarche de **narrowing progressif** (du plus haut niveau — curl — vers le plus bas niveau — tcpdump) est la méthode attendue à l'examen, que l'outil final utilisé soit Hubble/Prometheus (UI) ou ces commandes brutes.
