# Gestion du temps — le facteur d'échec n°1 rapporté par 2 candidats beta sur 2

## Les faits rapportés
- **jkroepke** : "Time was the hardest part for me... Trouble with the in-browser desktop environment at the beginning also cost a few minutes."
- **Second candidat (LinkedIn)** : "je n'ai pas eu le temps de terminer l'ensemble des questions dans le temps imparti."

**2 candidats sur 2 rapportent le temps comme LE facteur limitant.** C'est le signal le plus fiable de tout ce dossier de préparation.

## Règles à appliquer dès l'entraînement

1. **Ne jamais perfectionner une tâche.** Résous, valide, passe à la suivante. Si une tâche prend plus de temps que prévu, marque-la et reviens-y en fin de session s'il reste du temps.
2. **Budget indicatif par domaine (sur 2h, ~15-20 tâches estimées) :**
   - Core Infra & CNI (15%) → ~18 min
   - Service Networking & DNS (25%) → ~30 min
   - Advanced Traffic Management (20%) → ~24 min
   - Network Security & Policy (25%) → ~30 min
   - Observability (15%) → ~18 min
   - Réserve/imprévus → ~10-15 min (dont le "démarrage lent" de l'environnement rapporté par jkroepke)
3. **Navigation rapide = compétence à part entière.** Le retour de beta est explicite : "Fast navigation through Kubernetes resources, documentation, and kubectl matters almost as much as knowing the concepts." Entraîne-toi à :
   - Utiliser des alias (`alias k=kubectl`)
   - Avoir l'auto-complétion kubectl activée
   - Savoir chercher rapidement dans la doc officielle (Ctrl+F sur les bonnes pages, connaître l'URL directe des pages clés)
4. **Ne pas mémoriser, comprendre.** Cité explicitement : "Documentation matters more than memorization... memorizing complete YAML structures was not especially useful... enough context was available to identify the required resource, together with direct links to relevant documentation." Entraîne ta capacité à ADAPTER un exemple de la doc plutôt qu'à réciter du YAML par cœur.
5. **Anticipe un environnement inconnu au démarrage.** Prévois mentalement 5 minutes de "prise en main" de l'interface d'examen (bureau distant dans le navigateur) sans paniquer si ça semble lent au début.

## Checklist mentale avant de démarrer l'examen réel
- [ ] J'ai fait AU MOINS un passage complet de ce repo en conditions chronométrées
- [ ] Je sais retrouver en moins de 30 secondes les pages officielles : Gateway API, Cilium docs, Istio security, cert-manager
- [ ] Je connais par cœur les commandes de diagnostic de base (`kubectl get/describe/logs`, `cilium status`, `hubble observe`, `cilium bgp peers`) sans avoir à réfléchir à la syntaxe
- [ ] Je sais qu'il est normal/acceptable de ne pas finir 100% des tâches — prioriser les tâches à fort poids (domaines 25%) si le temps presse
