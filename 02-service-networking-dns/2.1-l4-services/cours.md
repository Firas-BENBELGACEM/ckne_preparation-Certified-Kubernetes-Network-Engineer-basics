# 2.1 Configuring L4 Services

## Résumé
Les **Services Kubernetes** jouent le rôle de load balancer L4 (virtual server) : ils exposent un groupe de pods (sélectionnés par labels) derrière une IP virtuelle stable, alors que les IP de pods individuels changent constamment.

**3 types principaux :**

### ClusterIP (défaut)
Expose le Service uniquement à l'intérieur du cluster.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: echoserver
spec:
  ports:
    - port: 80
      targetPort: 8080   # port réel du conteneur — piège classique si mal configuré
      protocol: TCP
  type: ClusterIP
  selector:
    app: echoserver
```

### NodePort
Ouvre le même port (range 30000-32767) sur TOUS les nodes, qui agissent comme passerelles NAT.
```bash
kubectl describe service node-port
# NodePort: 30618/TCP → NAT vers targetPort sur les pods sains
```

### LoadBalancer
S'appuie sur l'API du cloud provider (AWS ELB, GCP LB) pour créer un LB externe. Sur cluster on-premise, Cilium peut jouer ce rôle via **LB-IPAM** (assignation d'IP externe) combiné à BGP ou L2/ARP announcements pour la joignabilité.

**Piège n°1 à l'examen (confirmé par le retour beta jkroepke)** : un Service cassé a souvent simplement le mauvais `port` ou `targetPort`. Toujours vérifier :
```bash
kubectl get svc <name> -o yaml | grep -A3 ports
kubectl get endpointslices -l kubernetes.io/service-name=<name>
```

## Sources officielles
- Services : https://kubernetes.io/docs/concepts/services-networking/service/
- Service without selector : https://kubernetes.io/docs/concepts/services-networking/service/#services-without-selectors
