# CKNE Preparation (Certified Kubernetes Network Engineer preparation)

Dépôt de préparation à la certification **CKNE (Certified Kubernetes Network Engineer)** de la Linux Foundation / CNCF, organisé strictement selon le blueprint officiel :
https://training.linuxfoundation.org/certification/certified-kubernetes-network-engineer-ckne/

## Structure

Chaque sous-thème du blueprint a son propre dossier contenant 3 fichiers :
- `cours.md` — résumé théorique + liens vers les sources officielles
- `exercice.md` — lab pratique en conditions proches de l'examen (kind/k3d)
- `correction.md` — solution commentée, commandes de validation

```
01-core-infrastructure-cni/          (15%)
02-service-networking-dns/           (25%)
03-advanced-traffic-management/      (20%)
04-network-security-policy/          (25%)
05-observability/                    (15%)
06-exam-simulation/                  mock exams + gestion du temps
```

## Sources utilisées pour construire ce repo

**Officielles :**
- Blueprint CKNE : https://training.linuxfoundation.org/certification/certified-kubernetes-network-engineer-ckne/
- Kubernetes Networking Model : https://kubernetes.io/docs/concepts/cluster-administration/networking/
- CNI Spec : https://github.com/containernetworking/cni/blob/main/SPEC.md
- Services Networking : https://kubernetes.io/docs/concepts/services-networking/
- Network Policies : https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Gateway API : https://gateway-api.sigs.k8s.io/
- Cilium docs : https://docs.cilium.io/en/stable/
- Istio docs : https://istio.io/latest/docs/
- cert-manager docs : https://cert-manager.io/docs/
- Gateway API Inference Extension : https://gateway-api-inference-extension.sigs.k8s.io/

**eBook fourni (Isovalent — "Kubernetes Networking and Cilium: An Instruction Manual for the Network Engineer", Nico Vibert et al.)** — utilisé comme base pédagogique pour les analogies réseau traditionnel ↔ Kubernetes.

**Retours d'expérience de candidats beta (à prendre comme signal fort, pas comme certitude — la beta peut évoluer) :**
- jkroepke, "Preparing for CKNE: What I Learned From the Beta Exam" (17/09/2026) — https://github.com/jkroepke/blog/blob/article/ckne-beta-exam/content/posts/preparing-for-ckne-what-i-learned-from-the-beta-exam/index.md
- Post LinkedIn d'un second candidat beta (anonymisé dans ce repo) confirmant : installation/upgrade Cilium, policies L7 Cilium/Istio, Prometheus + Jaeger, remplacement de CNI (Flannel → Cilium) avec perturbation de service autorisée, Gateway API avec Envoy, inference routing.

**Ressources communautaires à explorer en complément (non vérifiées ligne à ligne par ce repo — croise toujours avec la doc officielle) :**
- https://github.com/DeanKS/ckne-labs
- https://github.com/Kyle-Law/ckne-study-notes
- https://github.com/zhengyu89/Kubernetes-Learning-Path
- https://github.com/skmahe1077/ckne-exam-lab/tree/main/labs
- https://github.com/Stivenavila/ckne-practice-lab
- https://supachai-j.github.io/ckne-course/
- Série Medium @kylelzk (CKNE exam prep 1.1 à 4.4 + labs Gateway API Inference Extension, DNS, CoreDNS, Egress Gateway, Network Policies, chiffrement) — https://medium.com/@kylelzk
- https://isovalent.com/labs/

## Les 2 signaux les plus importants à retenir avant de commencer

1. **D'après le retour de beta le plus détaillé disponible (jkroepke) : l'examen récompense la largeur plus que la mémorisation.** Ne pas essayer de mémoriser des CRD complètes ou du PromQL complexe — comprendre le problème, trouver la doc, adapter, valider, passer à la suite. Le troubleshooting semble plus piloté par UI (Prometheus/Jaeger préinstallés) que par les outils Linux bas niveau.
2. **La gestion du temps est le facteur d'échec n°1 rapporté par les deux candidats.** L'un des deux n'a pas fini toutes les questions. Chronomètre-toi systématiquement sur chaque exercice de ce repo.

## Comment utiliser ce repo

1. Pour chaque sous-thème, lis le `cours.md` (10-15 min max).
2. Fais l'exercice en conditions d'examen : **chronomètre-toi**, objectif 10-15 min par exercice simple, 20-25 min pour les plus complexes (Gateway API, multi-cluster, inference).
3. Compare avec `correction.md` seulement après avoir essayé.
4. Termine par `06-exam-simulation/` en conditions réelles (2h, mélange de tous les domaines).

## Prérequis techniques

```bash
# Outils à avoir installés
kind ou k3d
kubectl
helm
cilium CLI (https://github.com/cilium/cilium-cli)
hubble CLI
istioctl (pour les exercices Istio)
```
