# Mock Exam 1 — 2 heures, mélange de tous les domaines

Chronomètre-toi réellement. Ne consulte les corrections des chapitres qu'APRÈS avoir fini (ou que le temps soit écoulé).

## Tâche 1 (Domaine 1 — 15%, ~10 min)
Un cluster utilise Flannel. Remplace-le par Cilium en conservant le PodCIDR `10.244.0.0/16`. Une interruption de service est acceptée.

## Tâche 2 (Domaine 2 — 25%, ~12 min)
Un Service `payments` ne retourne aucun endpoint. Diagnostique et corrige (indice : ne présupposant pas la cause, vérifie labels, selector, et readiness des pods).

## Tâche 3 (Domaine 2 — 25%, ~15 min)
Crée un `Gateway` + `HTTPRoute` exposant deux Services (`catalog` sur `/catalog` et `orders` sur `/orders`) via un unique point d'entrée HTTP.

## Tâche 4 (Domaine 3 — 20%, ~12 min)
Configure une `CiliumEgressGatewayPolicy` pour que tout le trafic sortant des pods `team=finance` du namespace `default` ressorte avec l'IP source `10.99.99.99`.

## Tâche 5 (Domaine 3 — 20%, ~10 min)
Explique et configure (au choix) BGP ou L2 Announcements pour exposer un Service LoadBalancer, en justifiant ton choix selon un contexte donné : "cluster on-premise dans un rack avec switch ToR BGP-capable."

## Tâche 6 (Domaine 4 — 25%, ~15 min)
Écris une CiliumNetworkPolicy L7 n'autorisant que les requêtes `GET` sur `/health` depuis n'importe quel pod du namespace `monitoring` vers les pods `app=critical-service`.

## Tâche 7 (Domaine 4 — 25%, ~12 min)
Un Gateway avec listener HTTPS ne démarre pas (`PROGRAMMED=False`). Diagnostique la chaîne cert-manager → Secret → Gateway et corrige.

## Tâche 8 (Domaine 4 — 25%, ~10 min)
Configure `PeerAuthentication` en mode STRICT et une `AuthorizationPolicy` limitant `backend` à n'accepter que le service account `frontend-sa`.

## Tâche 9 (Domaine 5 — 15%, ~10 min)
Identifie, via Hubble ou Prometheus, la cause d'un taux d'erreur HTTP 5xx élevé sur un Service donné.

## Tâche 10 (Domaine 5 — 15%, ~8 min)
Utilise `hubble observe` pour produire un résumé du trafic rejeté (DROPPED) des 5 dernières minutes vers un Service donné, avec la raison de chaque drop.

**Total estimé : ~114 minutes sur 120 — marge volontairement serrée pour simuler la pression réelle rapportée par les 2 candidats beta.**
