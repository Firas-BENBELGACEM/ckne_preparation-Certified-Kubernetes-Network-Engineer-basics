# Correction 5.2

```bash
# 1. Accès à Jaeger
kubectl port-forward -n istio-system svc/tracing 16686:80 &
# Ouvrir http://localhost:16686

# 2. Génération de trafic
kubectl exec deploy/service-a -c nginx -- curl -m 2 -s http://service-b
kubectl exec deploy/service-b -c nginx -- curl -m 2 -s http://service-c
# (dans un vrai scénario d'examen, le trafic traverserait automatiquement la chaîne
# via l'application ; ici on simule manuellement pour l'exercice)

# 3. Recherche dans Jaeger UI
# Menu "Search" > sélectionner "service-a" dans le dropdown Service > Find Traces

# 4. Analyse d'une trace
# Cliquer sur une trace récente : la vue "waterfall" montre chaque span empilé avec sa durée.
# Le span le plus large horizontalement = celui qui consomme le plus de temps.

# 5. Réponse :
# Une lenteur RÉSEAU se manifeste par un ESPACE VIDE (gap) entre la fin d'un span parent et
# le début du span enfant suivant — c'est le temps "perdu" en transit qui n'est attribuable
# à aucun traitement applicatif mesuré.
# Une lenteur APPLICATIVE se manifeste par un span LUI-MÊME très large (le temps est DANS
# le traitement du service, pas entre deux services) — typiquement un appel base de données
# lent, un calcul CPU-intensif, etc.
```

**Point clé pour l'examen** : ne perds pas de temps à essayer de comprendre l'intégralité de l'instrumentation OpenTelemetry sous-jacente — le retour de beta est clair, l'objectif est de savoir **naviguer l'UI Jaeger déjà déployée** pour repérer visuellement où se situe un goulet d'étranglement, puis agir en conséquence.
